## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [mint and burn of PolygonERC20Wrapper](https://github.com/code-423n4/2021-12-amun-findings/issues/275) 

# Handle

pauliax


# Vulnerability details

## Impact
I am sorry if I incorrectly understood the intentions of contract PolygonERC20Wrapper but the comments and the code do not align here:

Function deposit:
  "Should handle deposit by minting the required amount for user. Make sure minting is done only by this function".
However, the code does not perform any minting and just transfers the underlying tokens to the user.

On the other hand, functions withdraw and withdrawTo state:
  "Should burn user's tokens" 
but it actually performs both, minting and burning:
```solidity
  _mint(recipient, amount);
  _burn(recipient, amount);
```

## Recommended Mitigation Steps
I was late to verify this with the sponsor, so make sure this is the intended behavior, and update comments to match the codebase.

