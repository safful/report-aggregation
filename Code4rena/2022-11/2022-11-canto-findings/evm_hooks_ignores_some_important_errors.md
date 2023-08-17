## Tags

- bug
- 2 (Med Risk)
- primary issue
- selected for report
- sponsor confirmed
- M-01

# [evm_hooks ignores some important errors](https://github.com/code-423n4/2022-11-canto-findings/issues/55) 

# Lines of code

https://github.com/code-423n4/2022-11-canto/blob/2733fdd1bee73a6871c6243f92a007a0b80e4c61/Canto/x/csr/keeper/evm_hooks.go#L49
https://github.com/code-423n4/2022-11-canto/blob/2733fdd1bee73a6871c6243f92a007a0b80e4c61/Canto/x/csr/keeper/evm_hooks.go#L101-L135


# Vulnerability details

## Impact
Some contracts and some Turnstile tokens (nfts) wll not be able to receive CSR fees forever. 

## Proof of Concept
In evm_hooks.go, the [PostTxProcessing](https://github.com/code-423n4/2022-11-canto/blob/2733fdd1bee73a6871c6243f92a007a0b80e4c61/Canto/x/csr/keeper/evm_hooks.go#L49) will call `h.processEvents(ctx, receipt)` to handle `Register` and `Assign` events from Turnstile contract first:
  ```
  h.processEvents(ctx, receipt)
  ```
Notice that the `processEvents` function does not return any error.

However, it is possible for [processEvents](https://github.com/code-423n4/2022-11-canto/blob/2733fdd1bee73a6871c6243f92a007a0b80e4c61/Canto/x/csr/keeper/evm_hooks.go#L101-L135) to encounter an error:

  ```
  func (h Hooks) processEvents(ctx sdk.Context, receipt *ethtypes.Receipt) {
    ...
    for _, log := range receipt.Logs {
      ...
      if log.Address == turnstileAddress {
        ...
        switch event.Name {
        case types.TurnstileEventRegister:
          err = h.k.RegisterEvent(ctx, log.Data)
        case types.TurnstileEventUpdate:
          err = h.k.UpdateEvent(ctx, log.Data)
        }
        if err != nil {
          h.k.Logger(ctx).Error(err.Error())
          return
        }
      }
    }
  }
  ```

According to the above implementation of `processEvents`, it will process all the events emitted by the transaction one by one. If one of them encounters an error, it will return directly without any error, and any subsequent unprocessed events will be ignored.

Suppose we have a transaction contains the following events(by contract calls):
1. Register C1 with token1
2. Register C2 with token2
3. Assign C3 with token1

If `RegisterEvent()` returns an error when handling the first event, then all of the events will not be handled because `processEvents()` will return after logging the error.
And `PostTxProcessing()` continues to execute normally because it is unaware of the error.

According to the current implementation of [`RegisterEvent()`](https://github.com/code-423n4/2022-11-canto/blob/main/Canto/x/csr/keeper/event_handler.go#L16) and [`UpdateEvent`](https://github.com/code-423n4/2022-11-canto/blob/main/Canto/x/csr/keeper/event_handler.go#L62), they are both easy to encounter an error. Like `register()` using a recipient that doesn't exist yet.

As a result, none of the C1, C2, C3 contracts will be able to recieve any CSR fee because they are not recorded in csr store.

Contrcts C1, C2, C3 will never be able to register for CSR because they are marked registered in Turnstile contract (evm store) and will be reverted by `onlyUnregistered` when calling `register()` or `assign()`.

And all other contracts calling `assign(token1)` or `assign(token2)` will enter the same state as C1/C2/C3, because the `assign()` will succeed in Turnstile contract but fail in [`UpdateEvent()`](https://github.com/code-423n4/2022-11-canto/blob/2733fdd1be/Canto/x/csr/keeper/event_handler.go#L75-L80) (because the store can not find token1 or token2):

  ```
	// Check if the NFT that is being updated exists in the CSR store
	nftID := event.TokenId.Uint64()
	csr, found := k.GetCSR(ctx, nftID)
	if !found {
		return sdkerrors.Wrapf(ErrNFTNotFound, "EventHandler::UpdateEvent the nft entered does not currently exist: %d", nftID)
	}
  ```

## Tools Used
VS Code

## Recommended Mitigation Steps

`processEvents()` should return the error it encounters, and `PostTxProcessing()` should return that error too.
