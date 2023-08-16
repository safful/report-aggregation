## Tags

- bug
- sponsor confirmed
- G (Gas Optimization)
- resolved

# [Repeated token transfers on deposits are unnecessary](https://github.com/code-423n4/2021-10-tempus-findings/issues/16) 

# Handle

TomFrench


# Vulnerability details

## Impact

Higher gas costs on transfers of tokens from user to TempusPool

## Proof of Concept

Following the flow of tokens from the user to their the `TempusPool contract`:

1. User calls `TempusController.depositBacking`, `TempusController` transfers user's tokens to itself and approves relevant `TempusPool`

https://github.com/code-423n4/2021-10-tempus/blob/63f7639aad08f2bba717830ed81e0649f7fc23ee/contracts/TempusController.sol#L412-L414

2. `TempusController` calls `TempusPool.deposit` which in turn transfers tokens from the `TempusController` and then invests them.

https://github.com/code-423n4/2021-10-tempus/blob/63f7639aad08f2bba717830ed81e0649f7fc23ee/contracts/TempusPool.sol#L178

This first transfer is then superfluous as the `TempusPool` trusts the `TempusController` (it's the only contract which may call `deposit`). We're then incurring the costs of 1 `transfer` and 1 `approve` unnecessarily.

## Recommended Mitigation Steps

As `TempusPool` trusts `TempusController`, `TempusController` can transfer the tokens directly to `TempusPool` and just tell it how much has been deposited.

L412-L414 of `TempusController.sol` would then be replaced with:
```
// Deposit to directly to targetPool
uint transferredYBT = yieldBearingToken.untrustedTransferFrom(msg.sender, targetPool, yieldTokenAmount);
```

