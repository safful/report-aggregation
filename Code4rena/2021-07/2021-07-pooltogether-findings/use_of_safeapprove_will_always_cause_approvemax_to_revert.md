## Tags

- bug
- 2 (Med Risk)
- mStableYieldSource
- sponsor confirmed

# [Use of safeApprove will always cause approveMax to revert](https://github.com/code-423n4/2021-07-pooltogether-findings/issues/47) 

# Handle

0xRajeev


# Vulnerability details

## Impact

Unlike SwappableYieldSource which uses safeIncreaseAllowance to increase the allowance to uint256.max, mStableYieldSource uses OpenZeppelin’s safeApprove() which has been documented as 1) Deprecated because of approve-like race condition and 2) To be used only for initial setting of allowance (current allowance == 0) or resetting to 0 because it reverts otherwise.

The usage here is intended to allow increase of allowance when it falls low similar to the documented usage in SwappableYieldSource. Using it for that scenario will not work as expected because it will always revert if current allowance is != 0. The initial allowance is already set as uint256.max in constructor. And once it gets reduced, it can never be increased using this function unless it is invoked when allowance is reduced completely to 0.

## Proof of Concept

https://github.com/pooltogether/pooltogether-mstable/blob/0bcbd363936fadf5830e9c48392415695896ddb5/contracts/yield-source/MStableYieldSource.sol#L60-L65

https://github.com/pooltogether/pooltogether-mstable/blob/0bcbd363936fadf5830e9c48392415695896ddb5/contracts/yield-source/MStableYieldSource.sol#L51

https://github.com/pooltogether/swappable-yield-source/blob/89cf66a3e3f8df24a082e1cd0a0e80d08953049c/contracts/SwappableYieldSource.sol#L135-L143

https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/blob/081776bf5fae2122bfda8a86d5369496adfdf959/contracts/token/ERC20/utils/SafeERC20Upgradeable.sol#L37-L57


## Tools Used

Manual Analysis

## Recommended Mitigation Steps

Use logic similar to SwappableYieldSource instead of using safeApprove().

