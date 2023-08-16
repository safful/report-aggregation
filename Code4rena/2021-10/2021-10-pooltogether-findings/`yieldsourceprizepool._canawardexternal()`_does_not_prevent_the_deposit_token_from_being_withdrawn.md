## Tags

- bug
- sponsor confirmed
- 0 (Non-critical)
- resolved

# [`YieldSourcePrizePool._canAwardExternal()` Does Not Prevent the Deposit Token From Being Withdrawn](https://github.com/code-423n4/2021-10-pooltogether-findings/issues/58) 

# Handle

leastwood


# Vulnerability details

## Impact

The `_canAwardExternal()` function is used to prevent the `onlyPrizeStrategy` role from moving a yield source's deposit tokens. However, the `YieldSourcePrizePool.sol` contract only restricts the movement of tokens from the `yieldSource` address instead of the actual deposit token. As a result, the `onlyOwner` role could escalate its role by calling `PrizePool.setPrizeStrategy()` and setting the prize strategy to its own address. Once it has taken over this role, they could effectively transfer out the yield source's deposit tokens, thereby draining the contract. 

This is in direct contrast to PoolTogether's ethos, whereby their docs state that the multisig account used to represent the `onlyOwner` role has no custody over deposited assets.

## Proof of Concept

https://v4.docs.pooltogether.com/protocol/reference/launch-architecture#progressive-decentralization
https://github.com/pooltogether/v4-core/blob/master/contracts/prize-pool/YieldSourcePrizePool.sol#L55-L57
https://github.com/pooltogether/v4-core/blob/master/contracts/prize-pool/PrizePool.sol#L329-L343
https://github.com/pooltogether/v4-core/blob/master/contracts/prize-pool/PrizePool.sol#L228-L236
https://github.com/pooltogether/v4-core/blob/master/contracts/prize-pool/PrizePool.sol#L300-L302

## Tools Used

Manual code review

## Recommended Mitigation Steps

Consider updating the `YieldSourcePrizePool._canAwardExternal()` function to restrict the prize strategy from withdrawing `yieldSource.depositToken()` instead.

