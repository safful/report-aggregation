## Tags

- bug
- 3 (High Risk)
- primary issue
- selected for report
- sponsor confirmed
- H-02

# [A registered contract won't earn fees if `_recipient` is a fresh address](https://github.com/code-423n4/2022-11-canto-findings/issues/93) 

# Lines of code

https://github.com/code-423n4/2022-11-canto/blob/2733fdd1bee73a6871c6243f92a007a0b80e4c61/Canto/x/csr/keeper/event_handler.go#L31-L33


# Vulnerability details

## Impact
Users might fall victims of a false positive: if they use a fresh account as an NFT recipient during contract registration, the transaction won't revert, but the registered contract will never earn fees for the token holder. And since a contract can be registered only once, there won't be a way for affected users to re-register contracts and start earning fees. This can affect both big and smaller project that register their contracts with the Turnstile contract: the only condition for the bug to happen is that the recipient address that's used during registration is a fresh address (i.e. an address that hasn't been used yet).
## Proof of Concept
The `register` function allows the calling contract to specify the address that will receive the freshly minted NFT ([Turnstile.sol#L86](https://github.com/code-423n4/2022-11-canto/blob/2733fdd1bee73a6871c6243f92a007a0b80e4c61/CIP-001/src/Turnstile.sol#L86)):
```solidity
function register(address _recipient) public onlyUnregistered returns (uint256 tokenId) {
    address smartContract = msg.sender;

    if (_recipient == address(0)) revert InvalidRecipient();

    tokenId = _tokenIdTracker.current();
    _mint(_recipient, tokenId);
    _tokenIdTracker.increment();

    emit Register(smartContract, _recipient, tokenId);

    feeRecipient[smartContract] = NftData({
        tokenId: tokenId,
        registered: true
    });
}
```

A recipient address can be any address besides the zero address. However, on the consensus layer, there's a stricter requirement ([event_handler.go#L31-L33](https://github.com/code-423n4/2022-11-canto/blob/2733fdd1bee73a6871c6243f92a007a0b80e4c61/Canto/x/csr/keeper/event_handler.go#L31-L33)): a recipient address cannot be a *fresh account*, that is an address that:
- hasn't ever received native coins;
- hasn't ever sent a transaction;
- hasn't ever had contract code.

While, on the application layer, calling the `register` function with a fresh address will succeed, on the consensus layer a contract won't be registered. When a `Register` event is processed on the consensus layer, there's a check that requires that the recipient address is an *existing account* in the state database ([event_handler.go#L31-L33](https://github.com/code-423n4/2022-11-canto/blob/2733fdd1bee73a6871c6243f92a007a0b80e4c61/Canto/x/csr/keeper/event_handler.go#L31-L33)):
```go
// Check that the receiver account  exists in the evm store
if acct := k.evmKeeper.GetAccount(ctx, event.Recipient); acct == nil {
  return sdkerrors.Wrapf(ErrNonexistentAcct, "EventHandler::RegisterEvent account does not exist: %s", event.Recipient)
}
```

If the recipient account doesn't exist, the function will return, but the register transaction won't revert (errors during the events processing doesn't result in a revert: [evm_hooks.go#L123-L132](https://github.com/code-423n4/2022-11-canto/blob/2733fdd1bee73a6871c6243f92a007a0b80e4c61/Canto/x/csr/keeper/evm_hooks.go#L123-L132), [evm_hooks.go#L49](https://github.com/code-423n4/2022-11-canto/blob/2733fdd1bee73a6871c6243f92a007a0b80e4c61/Canto/x/csr/keeper/evm_hooks.go#L49)).

The `GetAccount` function above returns `nil` when an address doesn't exist in the state database. To see this, we need to unwind the `GetAccount` execution:
1. the `GetAccount` is called on an `evmKeeper` ([event_handler.go#L31](https://github.com/code-423n4/2022-11-canto/blob/2733fdd1bee73a6871c6243f92a007a0b80e4c61/Canto/x/csr/keeper/event_handler.go#L31)):
    ```go
    if acct := k.evmKeeper.GetAccount(ctx, event.Recipient); acct == nil {
      return sdkerrors.Wrapf(ErrNonexistentAcct, "EventHandler::RegisterEvent account does not exist: %s", event.Recipient)
    }
    ```
1. `evmKeeper` is set during the CSR Keeper initialization ([keeper.go#L27](https://github.com/code-423n4/2022-11-canto/blob/2733fdd1bee73a6871c6243f92a007a0b80e4c61/Canto/x/csr/keeper/keeper.go#L27)):
    ```go
    func NewKeeper(
      cdc codec.BinaryCodec,
      storeKey sdk.StoreKey,
      ps paramtypes.Subspace,
      accountKeeper types.AccountKeeper,
      evmKeeper types.EVMKeeper,
      bankKeeper types.BankKeeper,
      FeeCollectorName string,
    ) Keeper {
      // set KeyTable if it has not already been set
      if !ps.HasKeyTable() {
        ps = ps.WithKeyTable(types.ParamKeyTable())
      }

      return Keeper{
        storeKey:         storeKey,
        cdc:              cdc,
        paramstore:       ps,
        accountKeeper:    accountKeeper,
        evmKeeper:        evmKeeper,
        bankKeeper:       bankKeeper,
        FeeCollectorName: FeeCollectorName,
      }
    }
    ```
1. the CSR Keeper is initialized during the main app initialization ([app.go#L473-L478](https://github.com/code-423n4/2022-11-canto/blob/2733fdd1bee73a6871c6243f92a007a0b80e4c61/Canto/app/app.go#L473-L478)), this is also when the EVM Keeper is initialized ([app.go#L409-L413](https://github.com/code-423n4/2022-11-canto/blob/2733fdd1bee73a6871c6243f92a007a0b80e4c61/Canto/app/app.go#L409-L413)):
    ```go
    app.EvmKeeper = evmkeeper.NewKeeper(
      appCodec, keys[evmtypes.StoreKey], tkeys[evmtypes.TransientKey], app.GetSubspace(evmtypes.ModuleName),
      app.AccountKeeper, app.BankKeeper, &stakingKeeper, app.FeeMarketKeeper,
      tracer,
    )
    ```
1. the EVM Keeper is implemented and imported from Ethermint ([keeper.go#L67](https://github.com/evmos/ethermint/blob/f7f1e1c18c64c912e7b5d524ae710e542b47e02a/x/evm/keeper/keeper.go#L67));
1. here's the `GetAccount` function ([statedb.go#L25](https://github.com/evmos/ethermint/blob/f7f1e1c18c64c912e7b5d524ae710e542b47e02a/x/evm/keeper/statedb.go#L25)):
    ```go
    func (k *Keeper) GetAccount(ctx sdk.Context, addr common.Address) *statedb.Account {
      acct := k.GetAccountWithoutBalance(ctx, addr)
      if acct == nil {
        return nil
      }

      acct.Balance = k.GetBalance(ctx, addr)
      return acct
    }
    ```
1. the `GetAccountWithoutBalance` function calls `GetAccount` on `accountKeeper` ([keeper.go#L255-L258](https://github.com/evmos/ethermint/blob/f7f1e1c18c64c912e7b5d524ae710e542b47e02a/x/evm/keeper/keeper.go#L255-L258)):
    ```go
    acct := k.accountKeeper.GetAccount(ctx, cosmosAddr)
    if acct == nil {
      return nil
    }
    ```
1. The Account Keeper is implemented in the Cosmos SDK ([account.go#L41-L49](https://github.com/cosmos/cosmos-sdk/blob/394f1b9478a8dc568d4bab079732932488b46704/x/auth/keeper/account.go#L41-L49)):
    ```go
    func (ak AccountKeeper) GetAccount(ctx sdk.Context, addr sdk.AccAddress) types.AccountI {
      store := ctx.KVStore(ak.storeKey)
      bz := store.Get(types.AddressStoreKey(addr))
      if bz == nil {
        return nil
      }

      return ak.decodeAccount(bz)
    }
    ```
1. it basically reads an account from the store passed in the context object ([context.go#L280-L282](https://github.com/cosmos/cosmos-sdk/blob/394f1b9478a8dc568d4bab079732932488b46704/types/context.go#L280-L282));
1. in the Account Keeper, there's also `SetAccount` function ([account.go#L72](https://github.com/cosmos/cosmos-sdk/blob/394f1b9478a8dc568d4bab079732932488b46704/x/auth/keeper/account.go#L72)), and it's called in Ethermint by the EVM Keeper ([statedb.go#L126](https://github.com/evmos/ethermint/blob/f7f1e1c18c64c912e7b5d524ae710e542b47e02a/x/evm/keeper/statedb.go#L126));
1. the EVM Keeper's `SetAccount` is called when transaction changes are committed to the state database ([statedb.go#L449](https://github.com/evmos/ethermint/blob/f7f1e1c18c64c912e7b5d524ae710e542b47e02a/x/evm/statedb/statedb.go#L449));
1. the state database is a set of state objects, where keys are account addresses and values are accounts themselves;
1. the `getOrNewStateObject` function initializes new state objects ([statedb.go#L221-L227](https://github.com/evmos/ethermint/blob/f7f1e1c18c64c912e7b5d524ae710e542b47e02a/x/evm/statedb/statedb.go#L221-L227));
1. `getOrNewStateObject` is only called by these functions: `AddBalance`, `SubBalance`, `SetNonce`, `SetCode`, `SetState` ([statedb.go#L290-L328](https://github.com/evmos/ethermint/blob/f7f1e1c18c64c912e7b5d524ae710e542b47e02a/x/evm/statedb/statedb.go#L290-L328)).

Thus, a new account object in the state database is only created when an address receives native coins, sends a transaction (which increases the nonce), or when contract code is deployed at it.

### Example Exploit Scenario
1. Alice deploys a smart contract that attracts a lot of users.
1. Alice registers the contract in Turnstile. As a recipient contract for the NFT, Alice decides to use a dedicated address that hasn't been used for anything else before (hasn't received coins, hasn't sent a transaction, etc.).
1. The `register` function call succeeds and the Alice's contract gets registered in Turnstile.
1. However, due to the "only existing recipient account" check on the consensus layer, Alice's contract wasn't registered on the consensus layer and doesn't earn fees.
1. Since `register` and `assign` can only be called once (due to the `onlyUnregistered` modifier), Alice cannot re-register her contract. She can transfer the NFT to a different address, however this won't make the contract registered on the consensus layer and the owner of the NFT will never receive fees.
## Tools Used
Manual review
## Recommended Mitigation Steps
Consider removing the "only existing recipient account" check in the `RegisterEvent` handler since it creates a discrepancy between the application and the consensus layers. Otherwise, if it's mandatory that receiver addresses are not fresh, consider returning an error in the `PostTxProcessing` hook (which will revert a transaction) if there was an error during events processing.