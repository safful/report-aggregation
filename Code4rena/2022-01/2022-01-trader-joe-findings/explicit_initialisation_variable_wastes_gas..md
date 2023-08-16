## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Explicit initialisation variable wastes gas.](https://github.com/code-423n4/2022-01-trader-joe-findings/issues/87) 

# Handle

TomFrenchBlockchain


# Vulnerability details

## Impact
gas costs

## Proof of Concept

LaunchEvent has an explicit `initialized` variable which is in storage.
https://github.com/code-423n4/2022-01-trader-joe/blob/a1579f6453bc4bf9fb0db9c627beaa41135438ed/contracts/LaunchEvent.sol#L228

To save an SSTORE we could just check that `_auctionStart > 0` as this is a sufficient check for initialisation. If a getter is needed then a function like the below could be added.

```
function initialized() external view returns bool {
  return auctionStart > 0;
}
```

## Recommended Mitigation Steps

As above.

