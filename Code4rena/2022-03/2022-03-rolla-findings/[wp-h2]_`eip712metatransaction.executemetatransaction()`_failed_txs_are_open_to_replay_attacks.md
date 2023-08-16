## Tags

- bug
- 3 (High Risk)
- resolved
- sponsor confirmed

# [[WP-H2] `EIP712MetaTransaction.executeMetaTransaction()` failed txs are open to replay attacks](https://github.com/code-423n4/2022-03-rolla-findings/issues/45) 

# Lines of code

https://github.com/code-423n4/2022-03-rolla/blob/efe4a3c1af8d77c5dfb5ba110c3507e67a061bdd/quant-protocol/contracts/utils/EIP712MetaTransaction.sol#L86


# Vulnerability details

Any transactions that fail based on some conditions that may change in the future are not safe to be executed again later (e.g. transactions that are based on others actions, or time-dependent etc).

In the current implementation, once the low-level call is failed, the whole tx will be reverted and so that `_nonces[metaAction.from]` will remain unchanged.

As a result, the same tx can be replayed by anyone, using the same signature.

https://github.com/code-423n4/2022-03-rolla/blob/efe4a3c1af8d77c5dfb5ba110c3507e67a061bdd/quant-protocol/contracts/utils/EIP712MetaTransaction.sol#L86

```solidity
    function executeMetaTransaction(
        MetaAction memory metaAction,
        bytes32 r,
        bytes32 s,
        uint8 v
    ) external payable returns (bytes memory) {
        require(
            _verify(metaAction.from, metaAction, r, s, v),
            "signer and signature don't match"
        );

        uint256 currentNonce = _nonces[metaAction.from];

        // intentionally allow this to overflow to save gas,
        // and it's impossible for someone to do 2 ^ 256 - 1 meta txs
        unchecked {
            _nonces[metaAction.from] = currentNonce + 1;
        }

        // Append the metaAction.from at the end so that it can be extracted later
        // from the calling context (see _msgSender() below)
        (bool success, bytes memory returnData) = address(this).call(
            abi.encodePacked(
                abi.encodeWithSelector(
                    IController(address(this)).operate.selector,
                    metaAction.actions
                ),
                metaAction.from
            )
        );

        require(success, "unsuccessful function call");
        emit MetaTransactionExecuted(
            metaAction.from,
            payable(msg.sender),
            currentNonce
        );
        return returnData;
    }
```

See also the implementation of OpenZeppelin's `MinimalForwarder`:

https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v4.5.0/contracts/metatx/MinimalForwarder.sol#L42-L66

### PoC

Given:

- The collateral is USDC;
- Alice got `10,000 USDC` in the wallet.

1. Alice submitted a MetaTransaction to `operate()` and `_mintOptionsPosition()` with `10,000 USDC`;
2. Before the MetaTransaction get executed, Alice sent `1,000 USDC` to Bob;
3. The MetaTransaction submited by Alice in step 1 get executed but failed;
4. A few days later, Bob sent `1,000 USDC` to Alice;
5. The attacker can replay the MetaTransaction failed to execute at step 3 and succeed.

Alice's `10,000 USDC` is now been spent unexpectedly against her will and can potentially cause fund loss depends on the market situation.

### Recommendation

Failed txs should still increase the nonce.

While implementating the change above, consider adding one more check to require sufficient gas to be paid, to prevent "insufficient gas griefing attack" as described in [this article](https://ipfs.io/ipfs/QmbbYTGTeot9ic4hVrsvnvVuHw4b5P7F5SeMSNX9TYPGjY/blog/ethereum-gas-dangers/).

