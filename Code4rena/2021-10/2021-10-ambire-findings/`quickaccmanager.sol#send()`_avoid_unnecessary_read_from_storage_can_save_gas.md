## Tags

- bug
- sponsor confirmed
- G (Gas Optimization)
- resolved

# [`QuickAccManager.sol#send()` Avoid unnecessary read from storage can save gas](https://github.com/code-423n4/2021-10-ambire-findings/issues/25) 

# Handle

WatchPug


# Vulnerability details

In `QuickAccManager.sol#send()`, `nonces[address(identity)]` is being read 2 times (1st at L58, 2nd at L64), the second read is unnecessary, cache it in the stack at the first read can save some gas.

https://github.com/code-423n4/2021-10-ambire/blob/bc01af4df3f70d1629c4e22a72c19e6a814db70d/contracts/wallet/QuickAccManager.sol#L55-L67

```solidity=55{58,64}
function send(Identity identity, QuickAccount calldata acc, DualSig calldata sigs, Identity.Transaction[] calldata txns) external {
    bytes32 accHash = keccak256(abi.encode(acc));
    require(identity.privileges(address(this)) == accHash, 'WRONG_ACC_OR_NO_PRIV');
    uint initialNonce = nonces[address(identity)];
    // Security: we must also hash in the hash of the QuickAccount, otherwise the sig of one key can be reused across multiple accs
    bytes32 hash = keccak256(abi.encode(
        address(this),
        block.chainid,
        accHash,
        nonces[address(identity)]++,
        txns,
        sigs.isBothSigned
    ));
```

### Recommendation

Change to:

```solidity=55{58,64}
function send(Identity identity, QuickAccount calldata acc, DualSig calldata sigs, Identity.Transaction[] calldata txns) external {
    bytes32 accHash = keccak256(abi.encode(acc));
    require(identity.privileges(address(this)) == accHash, 'WRONG_ACC_OR_NO_PRIV');
    uint initialNonce = nonces[address(identity)]++;
    // Security: we must also hash in the hash of the QuickAccount, otherwise the sig of one key can be reused across multiple accs
    bytes32 hash = keccak256(abi.encode(
        address(this),
        block.chainid,
        accHash,
        initialNonce,
        txns,
        sigs.isBothSigned
    ));
```

