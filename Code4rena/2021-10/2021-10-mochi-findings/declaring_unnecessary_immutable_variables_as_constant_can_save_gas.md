## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Declaring unnecessary immutable variables as constant can save gas](https://github.com/code-423n4/2021-10-mochi-findings/issues/117) 

# Handle

WatchPug


# Vulnerability details

In `MochiProfileV0.sol`, `secPerYear` is defined as an immutable variable while it's not configured as a parameter of the constructor. Thus, it can be declared as constant to save gas.

https://github.com/code-423n4/2021-10-mochi/blob/8458209a52565875d8b2cefcb611c477cefb9253/projects/mochi-core/contracts/profile/MochiProfileV0.sol#L23-L28

```solidity=23
uint256 public immutable secPerYear;

uint256 public override delay;

constructor(address _engine) {
    secPerYear = 31536000;
```

### Recommendation

Change to:

```solidity=23
uint256 public constant SEC_PER_YEAR = 31536000;
```

