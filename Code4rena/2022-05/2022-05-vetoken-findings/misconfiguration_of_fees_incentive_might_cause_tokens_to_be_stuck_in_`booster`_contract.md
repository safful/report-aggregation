## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Misconfiguration of Fees Incentive Might Cause Tokens To Be Stuck In `Booster` Contract](https://github.com/code-423n4/2022-05-vetoken-findings/issues/215) 

# Lines of code

https://github.com/code-423n4/2022-05-vetoken/blob/2d7cd1f6780a9bcc8387dea8fecfbd758462c152/contracts/Booster.sol#L193
https://github.com/code-423n4/2022-05-vetoken/blob/2d7cd1f6780a9bcc8387dea8fecfbd758462c152/contracts/Booster.sol#L576


# Vulnerability details

## Proof-of-Concept

The `Booster.setFeeInfo` function is responsible for setting the allocation of gauge fees between lockers and $VE3D stakers.  `lockFeesIncentive` and `stakerLockFeesIncentive` should add up to `10000` , which is equivalent to `100%`.

However, there is no validation check to ensure that that `_lockFeesIncentive` and `_stakerLockFeesIncentive` add up to `10000`. Thus, it entirely depends on the developer to get these two values right.

As such, it is possible to set `lockFeesIncentive + takerLockFeesIncentive` to be less than `100%`. This might happen due to human error. For instance, a typo (forget a few zero) or newly joined developer might not be aware of the fee denomination and called `setFeeInfo(40, 60)` instead of `setFeeInfo(4000, 6000)`.

[https://github.com/code-423n4/2022-05-vetoken/blob/2d7cd1f6780a9bcc8387dea8fecfbd758462c152/contracts/Booster.sol#L193](https://github.com/code-423n4/2022-05-vetoken/blob/2d7cd1f6780a9bcc8387dea8fecfbd758462c152/contracts/Booster.sol#L193)

```solidity
uint256 public constant FEE_DENOMINATOR = 10000;

// Set reward token and claim contract, get from Curve's registry
function setFeeInfo(uint256 _lockFeesIncentive, uint256 _stakerLockFeesIncentive) external {
    require(msg.sender == feeManager, "!auth");

    lockFeesIncentive = _lockFeesIncentive;
    stakerLockFeesIncentive = _stakerLockFeesIncentive;
    ..SNIP..
}
```

Assume that `setFeeInfo(40, 60)` is called instead of of `setFeeInfo(4000, 6000)`, only `1%` of the fee collected will be transferred to the users and the remaining `99%` of the fee collected will be stuck in the `Booster` contract.

[https://github.com/code-423n4/2022-05-vetoken/blob/2d7cd1f6780a9bcc8387dea8fecfbd758462c152/contracts/Booster.sol#L576](https://github.com/code-423n4/2022-05-vetoken/blob/2d7cd1f6780a9bcc8387dea8fecfbd758462c152/contracts/Booster.sol#L576)

```solidity
function earmarkFees() external returns (bool) {
    //claim fee rewards
    IStaker(staker).claimFees(feeDistro, feeToken);
    //send fee rewards to reward contract
    uint256 _balance = IERC20(feeToken).balanceOf(address(this));

    uint256 _lockFeesIncentive = _balance.mul(lockFeesIncentive).div(FEE_DENOMINATOR);
    uint256 _stakerLockFeesIncentive = _balance.mul(stakerLockFeesIncentive).div(
        FEE_DENOMINATOR
    );
    if (_lockFeesIncentive > 0) {
        IERC20(feeToken).safeTransfer(lockFees, _lockFeesIncentive);
        IRewards(lockFees).queueNewRewards(_lockFeesIncentive);
    }
    if (_stakerLockFeesIncentive > 0) {
        IERC20(feeToken).safeTransfer(stakerLockRewards, _stakerLockFeesIncentive);
        IRewards(stakerLockRewards).queueNewRewards(feeToken, _stakerLockFeesIncentive);
    }
    return true;
}
```

### Can we retrieve or "save" the tokens stuck in `Booster` contract?

Any veAsset (e.g. CRV, ANGLE) sitting on the `Booster` contract is claimable. However, in this case, the `feeToken` is likely not a veAsset, thus the remaining gauge fee will be stuck in the `Booster` contract perpetually. For instance, in Curve, the gauge fee is paid out in 3CRV, the LP token for the TriPool. ([Source](https://resources.curve.fi/crv-token/understanding-crv#staking-trading-fees))

## Impact

Users will lost their gauge fee if this happens.

## Recommended Mitigation Steps

Implement validation check to ensure that `lockFeesIncentive` and `takerLockFeesIncentive` add up to 100% to eliminate any risk of misconfiguration.

```solidity
uint256 public constant FEE_DENOMINATOR = 10000;

// Set reward token and claim contract, get from Curve's registry
function setFeeInfo(uint256 _lockFeesIncentive, uint256 _stakerLockFeesIncentive) external {
    require(msg.sender == feeManager, "!auth");
    require(_lockFeesIncentive + _stakerLockFeesIncentive == FEE_DENOMINATOR, "Invalid fees");

    lockFeesIncentive = _lockFeesIncentive;
    stakerLockFeesIncentive = _stakerLockFeesIncentive;
    ..SNIP..
}
```

