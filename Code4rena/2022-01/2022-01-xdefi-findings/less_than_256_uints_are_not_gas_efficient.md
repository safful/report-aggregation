## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Less than 256 uints are not gas efficient](https://github.com/code-423n4/2022-01-xdefi-findings/issues/97) 

# Handle

defsec


# Vulnerability details

## Impact

Lower than uint256 size storage instance variables are actually less gas efficient. E.g. using uint32 does not give any efficiency, actually, it is the opposite as EVM operates on default of 256-bit values so uint32 is more expensive in this case as it needs a conversion. It only gives improvements in cases where you can pack variables together, e.g. structs.

## Proof of Concept

1. Navigate to the following contracts.

```
https://github.com/XDeFi-tech/xdefi-distribution/blob/master/contracts/XDEFIDistribution.sol#L301
```

2. Expiry value is just used for the comparison with the block.timestamp.

## Tools Used

None

## Recommended Mitigation Steps

Consider to review all uint types. Change them with uint256 If the integer is not necessary to present with uint32.

