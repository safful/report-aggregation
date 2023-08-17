## Tags

- bug
- 2 (Med Risk)
- primary issue
- selected for report
- sponsor confirmed
- M-02

# [`PostTxProcessing` can revert user transactions not interacting with Turnstile](https://github.com/code-423n4/2022-11-canto-findings/issues/94) 

# Lines of code

https://github.com/code-423n4/2022-11-canto/blob/2733fdd1bee73a6871c6243f92a007a0b80e4c61/Canto/x/csr/keeper/evm_hooks.go#L63
https://github.com/code-423n4/2022-11-canto/blob/2733fdd1bee73a6871c6243f92a007a0b80e4c61/Canto/x/csr/keeper/evm_hooks.go#L75
https://github.com/code-423n4/2022-11-canto/blob/2733fdd1bee73a6871c6243f92a007a0b80e4c61/Canto/x/csr/keeper/evm_hooks.go#L81
https://github.com/code-423n4/2022-11-canto/blob/2733fdd1bee73a6871c6243f92a007a0b80e4c61/Canto/x/csr/keeper/evm_hooks.go#L88


# Vulnerability details

## Impact
Any transaction, even those that don't interact with the Turnstile contract, can be reverted by the `PostTxProcessing` hook if there was a CSR specific error. Thus, the CSR module can impair the behavior of smart contracts not related to the module.
## Proof of Concept
The `PostTxProcessing` is used by the keeper to register contracts with the CSR module and distribute gas fees to registered contracts ([evm_hooks.go#L41](https://github.com/code-423n4/2022-11-canto/blob/2733fdd1bee73a6871c6243f92a007a0b80e4c61/Canto/x/csr/keeper/evm_hooks.go#L41)). The hook can return an error while handling CSR specific operations:
- reading a CSR object from the storage ([evm_hooks.go#L61-L64](https://github.com/code-423n4/2022-11-canto/blob/2733fdd1bee73a6871c6243f92a007a0b80e4c61/Canto/x/csr/keeper/evm_hooks.go#L61-L64));
- sending gas fees to the module ([evm_hooks.go#L73-L76](https://github.com/code-423n4/2022-11-canto/blob/2733fdd1bee73a6871c6243f92a007a0b80e4c61/Canto/x/csr/keeper/evm_hooks.go#L73-L76));
- reading the address of Turnstile ([evm_hooks.go#L79-L82](https://github.com/code-423n4/2022-11-canto/blob/2733fdd1bee73a6871c6243f92a007a0b80e4c61/Canto/x/csr/keeper/evm_hooks.go#L79-L82));
- calling the `distributeFees` function ([evm_hooks.go#L85-L89](https://github.com/code-423n4/2022-11-canto/blob/2733fdd1bee73a6871c6243f92a007a0b80e4c61/Canto/x/csr/keeper/evm_hooks.go#L85-L89)).

In case any of these operations fails, the whole transaction will be reverted ([state_transition.go#L272-L278](https://github.com/evmos/ethermint/blob/37e394f309808d911b6c7d7eaf1e0c7a9cc1c70b/x/evm/keeper/state_transition.go#L272-L278)):
```go
if err = k.PostTxProcessing(tmpCtx, msg, receipt); err != nil {
  // If hooks return error, revert the whole tx.
  res.VmError = types.ErrPostTxProcessing.Error()
  k.Logger(ctx).Error("tx post processing failed", "error", err)
```

One example of when the hook can revert in normal circumstances is when the fees to be distributed are 0, which can be caused by a combination of low gas usage of a transaction, a small CSR share, and rounding (the fees are a share of the gas spent to execute a transaction: [evm_hooks.go#L66-L70](https://github.com/code-423n4/2022-11-canto/blob/2733fdd1bee73a6871c6243f92a007a0b80e4c61/Canto/x/csr/keeper/evm_hooks.go#L66-L70)). In this case, the `distributeFees` call will revert and will cause the whole transaction to be reverted as well ([evm_hooks.go#L86-L89](https://github.com/code-423n4/2022-11-canto/blob/2733fdd1bee73a6871c6243f92a007a0b80e4c61/Canto/x/csr/keeper/evm_hooks.go#L86-L89), [Turnstile.sol#L149](https://github.com/code-423n4/2022-11-canto/blob/2733fdd1bee73a6871c6243f92a007a0b80e4c61/CIP-001/src/Turnstile.sol#L149)).
## Tools Used
Manual review
## Recommended Mitigation Steps
In the `PostTxProcessing` hook, consider always logging errors and returning `nil` to avoid impairing user transactions. Also, consider logging a fatal error and exiting when the module cannot function due to an error.