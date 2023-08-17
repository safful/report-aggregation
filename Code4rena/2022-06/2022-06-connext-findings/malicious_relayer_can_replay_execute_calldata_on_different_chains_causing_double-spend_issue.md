## Tags

- bug
- 3 (High Risk)
- resolved
- sponsor confirmed

# [Malicious Relayer Can Replay Execute Calldata On Different Chains Causing Double-Spend Issue](https://github.com/code-423n4/2022-06-connext-findings/issues/144) 

# Lines of code

https://github.com/code-423n4/2022-06-connext/blob/4dd6149748b635f95460d4c3924c7e3fb6716967/contracts/contracts/core/connext/facets/BridgeFacet.sol#L411


# Vulnerability details

## Proof-of-Concept

> This issue is only applicable for fast-transfer. Slow transfer would not have this issue because of the built-in fraud-proof mechanism in Nomad.

First, the attacker will attempt to use Connext to send `1000 USDC` from Ethereum domain to Optimism domain.

Assume that the attacker happens to be a relayer on the relayer network utilised by Connext, and the attacker's relayer happens to be tasked to relay the above execute calldata to the Optimism's Connext [`BridgeFacet.execute`](https://github.com/code-423n4/2022-06-connext/blob/4dd6149748b635f95460d4c3924c7e3fb6716967/contracts/contracts/core/connext/facets/BridgeFacet.sol#L411) function.

Optimism's Connext `BridgeFacet.execute` received the execute calldata and observed within the calldata that it is a fast-transfer and Router A is responsible for providing the liquidity. It will then check that the router signature is valid, and proceed to transfer `1000 oUSDC` to attacker wallet (0x123456) in Optimism.

Next, attacker will update the `ExecuteArgs.local` within the execute calldata to a valid local representation of canonical token (USDC) used within Polygon. Attacker will then send the modified execute calldata to Polygon's Connext `BridgeFacet.execute` function. Assume that the same Router A is also providing liquidity in Polygon. The `BridgeFacet.execute` function checks that the router signature is valid, and proceed to transfer `1000 POS-USDC` to atttack wallet (0x123456) in Polygon. 

At this point, the attacker has `1000 oUSDC` and `1000 POS-USDC` in his wallets. When the nomad message arrives at Optimism, Router A can claim the `1000 oUSDC` back from Connext. However, Router A is not able to claim back any fund in Polygon.

Note that same wallet address exists on different chains. For instance, the wallet address on Etherum and Polygon is the same.

### Why changing the `ExecuteArgs.local` does not affect the router signature verification?

This is because the router signature is generated from the `transferId` + `pathLength` only, and these data are stored within the `CallParams params` within the `ExecuteArgs` struct.

[https://github.com/code-423n4/2022-06-connext/blob/4dd6149748b635f95460d4c3924c7e3fb6716967/contracts/contracts/core/connext/libraries/LibConnextStorage.sol#L77](https://github.com/code-423n4/2022-06-connext/blob/4dd6149748b635f95460d4c3924c7e3fb6716967/contracts/contracts/core/connext/libraries/LibConnextStorage.sol#L77)

```solidity
struct ExecuteArgs {
  CallParams params;
  address local; // local representation of canonical token
  address[] routers;
  bytes[] routerSignatures;
  uint256 amount;
  uint256 nonce;
  address originSender;
}
```

Within the [`BridgeFacet._executeSanityChecks`](https://github.com/code-423n4/2022-06-connext/blob/4dd6149748b635f95460d4c3924c7e3fb6716967/contracts/contracts/core/connext/facets/BridgeFacet.sol#L411) function, it will attempt to rebuild to `transferId` by calling the following code:

```solidity
// Derive transfer ID based on given arguments.
bytes32 transferId = _getTransferId(_args);
```

Within the [`BridgeFacet._getTransferId`](https://github.com/code-423n4/2022-06-connext/blob/4dd6149748b635f95460d4c3924c7e3fb6716967/contracts/contracts/core/connext/facets/BridgeFacet.sol#L719) function, we can see that the `s.tokenRegistry.getTokenId(_args.local)` will always return the canonical `tokenDomain` and `tokenId`. In our example, it will be `Ethereum` and  `USDC`. Therefore, as long as the attacker specify a valid local representation of canonical token on a chain, the `transferId` returned by `s.tokenRegistry.getTokenId(_args.local)` will always be the same across all domains. Thus, this allows the attacker to modify the `ExecuteArgs.local` and yet he could pass the router signature check.

[https://github.com/code-423n4/2022-06-connext/blob/4dd6149748b635f95460d4c3924c7e3fb6716967/contracts/contracts/core/connext/facets/BridgeFacet.sol#L719](https://github.com/code-423n4/2022-06-connext/blob/4dd6149748b635f95460d4c3924c7e3fb6716967/contracts/contracts/core/connext/facets/BridgeFacet.sol#L719)

```solidity
  function _getTransferId(ExecuteArgs calldata _args) private view returns (bytes32) {
    (uint32 tokenDomain, bytes32 tokenId) = s.tokenRegistry.getTokenId(_args.local);
    return _calculateTransferId(_args.params, _args.amount, _args.nonce, tokenId, tokenDomain, _args.originSender);
  }
```

## Impact 

Router liquidity would be drained by attacker, and affected router owner could not claim back their liquidity.

## Recommended Mitigation Steps

The security of the current Connext design depends on how secure or reliable the relayer is. If the relayer turns rouge or act against Connext, many serious consequences can happen.

The root cause is that the current design places enormous trust on the relayers to accurately and reliably to deliver calldata to the bridge in various domains. For instance, delivering of execute call data to `execute` function. There is an attempt to prevent message replay on a single domain, however, it does not prevent message replay across multiple domains. Most importantly, the Connext's bridge appears to have full trust on the calldata delivered by the relayer. However, the fact is that the calldata can always be altered by the relayer.

Consider a classic 0x off-chain ordering book protocol. A user will sign his order with his private key, and attach the signature to the order, and send the order (with signature) to the relayer network. If the relayer attempts to tamper the order message or signature, the decoded address will be different from the signer's address and this will be detected by 0x's Smart contract on-chain when processing the order. This ensures that the integrity of the message and signer can be enforced.

Per good security practice, relayer network should always be considered as a hostile environment/network. Therefore, it is recommended that similar approach could be taken with regards to passing execute calldata across domains/chains.

For instance, at a high level, the sequencer should sign the execute calldata with its private key, and attach the signature to the execute calldata. Then, submit the execute calldata (with signature) to the relayer network. When the bridge receives the execute calldata (with signature), it can verify if the decoded address matches the sequencer address to ensure that the calldata has not been altered. This will ensure the intergrity of the execute calldata and prevent any issue that arise due to unauthorised modification of calldata.

Additionally, the execute calldata should also have a field that correspond to the destination domain. The bridge that receives the execute calldata must verify that the execute calldata is intended for its domain, otherwise reject the calldata if it belongs to other domains. This also helps to prevent the attack mentioned earlier where same execute calldata can be accepted in different domains.

