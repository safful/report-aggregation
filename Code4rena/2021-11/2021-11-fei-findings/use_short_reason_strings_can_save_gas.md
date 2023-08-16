## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Use short reason strings can save gas](https://github.com/code-423n4/2021-11-fei-findings/issues/78) 

# Handle

WatchPug


# Vulnerability details

Every reason string takes at least 32 bytes.

Use short reason strings that fits in 32 bytes or it will become more expensive.

Instances include:

https://github.com/code-423n4/2021-11-fei/blob/add34324513b863f58e4ef7b3cd0c12d776dbb7f/contracts/TRIBERagequit.sol#L205-L208

```solidity
require(
            msg.sender == party0Timelock,
            "Only the timelock for party 0 may call this function"
        );
```

https://github.com/code-423n4/2021-11-fei/blob/add34324513b863f58e4ef7b3cd0c12d776dbb7f/contracts/TRIBERagequit.sol#L217-L220

```solidity
require(
            msg.sender == party1Timelock,
            "Only the timelock for party 1 may call this function"
        );
```

