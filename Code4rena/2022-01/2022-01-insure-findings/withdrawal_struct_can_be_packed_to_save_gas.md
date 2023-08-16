## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Withdrawal struct can be packed to save gas](https://github.com/code-423n4/2022-01-insure-findings/issues/27) 

# Handle

TomFrenchBlockchain


# Vulnerability details

## Impact
Detailed description of the impact of this finding.

## Proof of Concept

The `Withdrawal` struct in `IndexTemplate.sol` contains a timestamp and the amount of tokens which the user requests to withdraw.

https://github.com/code-423n4/2022-01-insure/blob/19d1a7819fe7ce795e6d4814e7ddf8b8e1323df3/contracts/IndexTemplate.sol#L81-L84

https://github.com/code-423n4/2022-01-insure/blob/19d1a7819fe7ce795e6d4814e7ddf8b8e1323df3/contracts/IndexTemplate.sol#L198

If we make the safe assumption that the user's balance does not exceed 2^192 then we can pack this struct into a single storage slot to save an SLOAD by changing the definition to:

```
struct Withdrawal {
    uint64 timestamp;
    uint192 amount;
}
```

## Recommended Mitigation Steps

As above

