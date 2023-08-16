## Tags

- bug
- 1 (Low Risk)
- resolved
- sponsor confirmed

# [Unsafe uint128 casting may overflow](https://github.com/code-423n4/2022-01-yield-findings/issues/112) 

# Handle

sirhashalot


# Vulnerability details

## Impact

The _calcRewardIntegral function casts intermediate reward values from uint256 to uint128 and vice versa several times. Because OpenZeppelin SafeCast is not used, casting from uint256 to uint128 may overflow if a large reward value is being calculate. This overflow could result in users receiving less rewards than they are owed.

## Proof of Concept

There are 4 uint128 casting operations and 2 uint256 casting operations [in the _calcRewardIntegral function of ConvexStakingWrapper.sol](https://github.com/code-423n4/2022-01-yield/blob/e946f40239b33812e54fafc700eb2298df1a2579/contracts/ConvexStakingWrapper.sol#L222-L253).

## Recommended Mitigation Steps

Because reward values are an important part of this protocol, use the OpenZeppelin SafeCast library to prevent unexpected overflows when casting. SafeMath and Solidity 0.8.* handles overflows for basic math operations but not for casting.

