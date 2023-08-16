## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [using += to save gas](https://github.com/code-423n4/2022-01-trader-joe-findings/issues/265) 

# Handle

rfa


# Vulnerability details

## Impact
expensive gas

## Proof of Concept
https://github.com/code-423n4/2022-01-trader-joe/blob/main/contracts/RocketJoeStaking.sol#L169-L172

## Tools Used

## Recommended Mitigation Steps
```
            accRJoePerShare  +=
            (rJoeReward * PRECISION) /
            joeSupply;
```

