## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Rearrange state variables](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/138) 

# Handle

pants


# Vulnerability details

## Impact
In BTCPoolDelegator contract, address state variables `future_owner` in lines 51 and bool state variable `is_killed` in line 55 should be placed one after another.
solidity keep storage in 32 bytes slot and can optimize multiple variables that are less than 32 bytes.
address is 20 bytes and bool is 1 byte, so it can be placed in one storage slot instead of two.

## Proof of Concept
Tested it on Remix, saves 50 gas per transaction

## Recommended Mitigation Steps
change
```
50    uint256 public future_admin_fee;
51    address public future_owner;
52
53    uint256 kill_deadline;
54    uint256 constant kill_deadline_dt = 2 * 30 * 86400;
54    bool is_killed;
```
to
```
50    uint256 public future_admin_fee;
51    uint256 constant kill_deadline_dt = 2 * 30 * 86400;
52
53    uint256 kill_deadline;
54    address public future_owner;
54    bool is_killed;
```

