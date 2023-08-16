## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [LinkedAddress struct can be packed to save an SSTORE](https://github.com/code-423n4/2021-12-sublime-findings/issues/72) 

# Handle

TomFrenchBlockchain


# Vulnerability details

## Impact

Gas costs on linking/unlinking addresses

## Proof of Concept

https://github.com/code-423n4/2021-12-sublime/blob/9df1b7c4247f8631647c7627a8da9bdc16db8b11/contracts/Verification/Verification.sol#L10-L13

The activation timestamp can be restricted to a `uint64` variable so that it shares a slot with the `masterAddress`. This will save an SSTORE and a substantial amount of gas.

## Recommended Mitigation Steps

As above.

