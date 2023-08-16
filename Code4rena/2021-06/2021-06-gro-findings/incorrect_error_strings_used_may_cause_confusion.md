## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed
- disagree with severity

# [Incorrect error strings used may cause confusion](https://github.com/code-423n4/2021-06-gro-findings/issues/58) 

# Handle

0xRajeev


# Vulnerability details

## Impact

Error strings used in require checks should accurately reflect the failing condition. Use of informative/accurate error messages helps troubleshoot exceptional conditions during transaction failures or unexpected behavior. Otherwise, it can be misleading and waste crucial time during exploits or emergency conditions. 

While the codebase has this correct in most places, there are a few places where there appears to be a copy/paste error:

Example 1: require(msg.sender == withdrawHandler || msg.sender == insurance, "depositStable: !depositHandler");

The error string should indicate “!withdrawHandler/insurance” instead of “!depositHandler”

Example 2: require(msg.sender == _controller().insurance(), "withdraw: !withdrawHandler/insurance");

The error string should only indicate “!insurance” instead of “!withdrawHandler/insurance”

## Proof of Concept

For reference, see Note 2 in OpenZeppelin's Audit of Compound Governor Bravo: https://blog.openzeppelin.com/compound-governor-bravo-audit/

https://github.com/code-423n4/2021-06-gro/blob/091660467fc8d13741f8aafcec80f1e8cf129a33/contracts/vaults/BaseVaultAdaptor.sol#L206-L211

https://github.com/code-423n4/2021-06-gro/blob/091660467fc8d13741f8aafcec80f1e8cf129a33/contracts/vaults/BaseVaultAdaptor.sol#L228

https://github.com/code-423n4/2021-06-gro/blob/091660467fc8d13741f8aafcec80f1e8cf129a33/contracts/pools/LifeGuard3Pool.sol#L162

https://github.com/code-423n4/2021-06-gro/blob/091660467fc8d13741f8aafcec80f1e8cf129a33/contracts/pools/LifeGuard3Pool.sol#L285

https://github.com/code-423n4/2021-06-gro/blob/091660467fc8d13741f8aafcec80f1e8cf129a33/contracts/Controller.sol#L405


## Tools Used

Manual Analysis

## Recommended Mitigation Steps

Check/fix error strings.

