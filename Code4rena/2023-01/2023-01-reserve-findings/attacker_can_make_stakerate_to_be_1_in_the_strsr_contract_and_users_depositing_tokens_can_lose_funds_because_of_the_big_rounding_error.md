## Tags

- bug
- 2 (Med Risk)
- satisfactory
- selected for report
- sponsor confirmed
- M-02

# [attacker can make stakeRate to be 1 in the StRSR contract and users depositing tokens can lose funds because of the big rounding error](https://github.com/code-423n4/2023-01-reserve-findings/issues/439) 

# Lines of code

https://github.com/reserve-protocol/protocol/blob/df7ecadc2bae74244ace5e8b39e94bc992903158/contracts/p1/StRSR.sol#L160-L188
https://github.com/reserve-protocol/protocol/blob/df7ecadc2bae74244ace5e8b39e94bc992903158/contracts/p1/StRSR.sol#L496-L530
https://github.com/reserve-protocol/protocol/blob/df7ecadc2bae74244ace5e8b39e94bc992903158/contracts/p1/StRSR.sol#L212-L237


# Vulnerability details

## Impact
code calculates amount of stake token and rsr token based on `stakeRate` and if `stakeRate` was near `1e18` then division error is small but attacker can cause `stakeRate` to be 1 and that can cause users to loss up to `1e18` token during stake and unstake.

## Proof of Concept
this is `init()` code:
```
    function init(
        IMain main_,
        string calldata name_,
        string calldata symbol_,
        uint48 unstakingDelay_,
        uint48 rewardPeriod_,
        uint192 rewardRatio_
    ) external initializer {
        require(bytes(name_).length > 0, "name empty");
        require(bytes(symbol_).length > 0, "symbol empty");
        __Component_init(main_);
        __EIP712_init(name_, "1");
        name = name_;
        symbol = symbol_;

        assetRegistry = main_.assetRegistry();
        backingManager = main_.backingManager();
        basketHandler = main_.basketHandler();
        rsr = IERC20(address(main_.rsr()));

        payoutLastPaid = uint48(block.timestamp);
        rsrRewardsAtLastPayout = main_.rsr().balanceOf(address(this));
        setUnstakingDelay(unstakingDelay_);
        setRewardPeriod(rewardPeriod_);
        setRewardRatio(rewardRatio_);

        beginEra();
        beginDraftEra();
    }
```
As you can see it sets the value of the `rsrRewardsAtLastPayout` as contract balance when contract is deployed.
This is `_payoutReward()` code:
```
    function _payoutRewards() internal {
        if (block.timestamp < payoutLastPaid + rewardPeriod) return;
        uint48 numPeriods = (uint48(block.timestamp) - payoutLastPaid) / rewardPeriod;

        uint192 initRate = exchangeRate();
        uint256 payout;

        // Do an actual payout if and only if stakers exist!
        if (totalStakes > 0) {
            // Paying out the ratio r, N times, equals paying out the ratio (1 - (1-r)^N) 1 time.
            // Apply payout to RSR backing
            // payoutRatio: D18 = FIX_ONE: D18 - FixLib.powu(): D18
            // Both uses of uint192(-) are fine, as it's equivalent to FixLib.sub().
            uint192 payoutRatio = FIX_ONE - FixLib.powu(FIX_ONE - rewardRatio, numPeriods);

            // payout: {qRSR} = D18{1} * {qRSR} / D18
            payout = (payoutRatio * rsrRewardsAtLastPayout) / FIX_ONE;
            stakeRSR += payout;
        }

        payoutLastPaid += numPeriods * rewardPeriod;
        rsrRewardsAtLastPayout = rsrRewards();

        // stakeRate else case: D18{qStRSR/qRSR} = {qStRSR} * D18 / {qRSR}
        // downcast is safe: it's at most 1e38 * 1e18 = 1e56
        // untestable:
        //      the second half of the OR comparison is untestable because of the invariant:
        //      if totalStakes == 0, then stakeRSR == 0
        stakeRate = (stakeRSR == 0 || totalStakes == 0)
            ? FIX_ONE
            : uint192((totalStakes * FIX_ONE_256 + (stakeRSR - 1)) / stakeRSR);

        emit RewardsPaid(payout);
        emit ExchangeRateSet(initRate, exchangeRate());
    }
```
As you can see it sets the value of the `stakeRate`  to `(totalStakes * FIX_ONE_256 + (stakeRSR - 1)) / stakeRSR`. 
So to exploit this attacker needs to perform this steps:
1. send `200 * 1e18` RSR tokens (18 is the precision) to the StRSR address before its deployment by watching mempool and front running. the deployment address is calculable before deployment.
2. function `init()` would get executed and would set `200 * 1e18` as `rsrRewardsAtLastPayout`.
3. then attacker would call `stake()` and stake 1 RSR token (1 wei) in the contract and the value of `stakeRSR` and `totalStakes` would be 1.
4. then attacker wait for `rewardPeriod` seconds and then call `payoutReward()` and code would pay rewards based on `rewardRatio` and `rsrRewardsAtLastPayout` and as `rewardRatio` is higher than 1% (default and normal mode) code would increase `stakeRate` more than `2 * 1e18` amount. and then code would set `stakeRate` as `totalStakes * FIX_ONE_256 + (stakeRSR - 1)) / stakeRSR = 1`.
5. then calls to `stake()` would cause users to lose up to `1e18` RSR tokens as code calculates stake amount as `newTotalStakes = (stakeRate * newStakeRSR) / FIX_ONE` and rounding error happens up to `FIX_ONE`. because the calculated stake amount is worth less than deposited rsr amount up to `1e18`.
6. attacker can still users funds by unstaking 1 token and receiving `1e18` RSR tokens. because of the rounding error in `unstake()`

so attacker can manipulate the `stakeRate` in contract deployment time with sandwich attack which can cause other users to lose funds because of the big rounding error.

## Tools Used
VIM

## Recommended Mitigation Steps
prevent early manipulation of the PPS