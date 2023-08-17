## Tags

- bug
- 2 (Med Risk)
- resolved
- sponsor confirmed

# [MINIMUM_LIQUIDITY checks missing - Bringing Liquidity below required min](https://github.com/code-423n4/2022-06-yieldy-findings/issues/48) 

# Lines of code

https://github.com/code-423n4/2022-06-yieldy/blob/main/src/contracts/LiquidityReserve.sol#L161


# Vulnerability details

## Impact
Whale who provided most liquidity to the contract can simply use removeLiquidity function and can remove all of his liquidity. This can leave the residual liquidity to be less than MINIMUM_LIQUIDITY which is incorrect

## Proof of Concept

1. Whale A provided initial liquidity plus more liquidity using enableLiquidityReserve and addLiquidity function

2. There are other small liquidity providers as well

3. Now Whale A decides to remove all the liquidity provided 

4. This means after liquidity removal the balance liquidity will even drop below MINIMUM_LIQUIDITY which is incorrect

## Recommended Mitigation Steps
Add below check

```
require(
            IERC20Upgradeable(stakingToken).balanceOf(address(this)) - MINIMUM_LIQUIDITY >=
                amountToWithdraw,
            "Not enough funds"
        );
```

