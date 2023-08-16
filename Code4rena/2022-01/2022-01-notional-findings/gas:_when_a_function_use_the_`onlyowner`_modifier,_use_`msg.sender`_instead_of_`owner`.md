## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Gas: When a function use the `onlyOwner` modifier, use `msg.sender` instead of `owner`](https://github.com/code-423n4/2022-01-notional-findings/issues/97) 

# Handle

Dravee


# Vulnerability details

## Impact
`msg.sender` costs 2 gas (CALLER opcode)
`owner` costs 100 gas (SLOAD opcode)
The `onlyOwner` modifier already checks that `msg.sender == owner`.

## Proof of Concept
Instances include:
```
contracts\sNOTE.sol:118:            payable(owner), // Owner will receive the NOTE and WETH
contracts\TreasuryManager.sol:108:        IERC20(token).safeTransfer(owner, amount);
```

## Tools Used
VS Code

## Recommended Mitigation Steps
When a function use the `onlyOwner` modifier, use `msg.sender` instead of `owner`

