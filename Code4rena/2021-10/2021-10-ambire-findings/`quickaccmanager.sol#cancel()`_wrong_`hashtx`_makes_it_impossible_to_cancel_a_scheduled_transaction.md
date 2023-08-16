## Tags

- bug
- sponsor confirmed
- 3 (High Risk)
- resolved

# [`QuickAccManager.sol#cancel()` Wrong `hashTx` makes it impossible to cancel a scheduled transaction](https://github.com/code-423n4/2021-10-ambire-findings/issues/1) 

# Handle

WatchPug


# Vulnerability details

In `QuickAccManager.sol#cancel()`, the `hashTx` to identify the transaction to be canceled is wrong. The last parameter is missing.

As a result, users will be unable to cancel a scheduled transaction.

https://github.com/code-423n4/2021-10-ambire/blob/bc01af4df3f70d1629c4e22a72c19e6a814db70d/contracts/wallet/QuickAccManager.sol#L91-L91

```solidity=81{91}
function cancel(Identity identity, QuickAccount calldata acc, uint nonce, bytes calldata sig, Identity.Transaction[] calldata txns) external {
    bytes32 accHash = keccak256(abi.encode(acc));
    require(identity.privileges(address(this)) == accHash, 'WRONG_ACC_OR_NO_PRIV');

    bytes32 hash = keccak256(abi.encode(CANCEL_PREFIX, address(this), block.chainid, accHash, nonce, txns, false));
    address signer = SignatureValidator.recoverAddr(hash, sig);
    require(signer == acc.one || signer == acc.two, 'INVALID_SIGNATURE');

    // @NOTE: should we allow cancelling even when it's matured? probably not, otherwise there's a minor grief
    // opportunity: someone wants to cancel post-maturity, and you front them with execScheduled
    bytes32 hashTx = keccak256(abi.encode(address(this), block.chainid, accHash, nonce, txns));
    require(scheduled[hashTx] != 0 && block.timestamp < scheduled[hashTx], 'TOO_LATE');
    delete scheduled[hashTx];

    emit LogCancelled(hashTx, accHash, signer, block.timestamp);
}
```

### Recommendation

Change to:

```solidity
bytes32 hashTx = keccak256(abi.encode(address(this), block.chainid, accHash, nonce, txns, false));
```

