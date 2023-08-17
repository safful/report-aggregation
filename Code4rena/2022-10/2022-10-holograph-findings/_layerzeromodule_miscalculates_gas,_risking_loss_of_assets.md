## Tags

- bug
- 3 (High Risk)
- resolved
- sponsor confirmed
- selected for report
- responded

# [ LayerZeroModule miscalculates gas, risking loss of assets](https://github.com/code-423n4/2022-10-holograph-findings/issues/445) 

# Lines of code

https://github.com/code-423n4/2022-10-holograph/blob/f8c2eae866280a1acfdc8a8352401ed031be1373/contracts/module/LayerZeroModule.sol#L431-L445


# Vulnerability details

## Description

Holograph gets it's cross chain messaging primitives through Layer Zero. To get pricing estimate, it uses the DstConfig price struct exposed in LZ's [RelayerV2](https://github.com/LayerZero-Labs/LayerZero/blob/main/contracts/RelayerV2.sol#L133)

The issue is that the important baseGas and gasPerByte configuration parameters, which are used to calculate a custom amount of gas for the destination LZ message, use the values that come from the *source* chain. This is in contrast to LZ which handles DstConfigs in a mapping keyed by chainID.  The encoded gas amount is described [here](https://layerzero.gitbook.io/docs/guides/advanced/relayer-adapter-parameters)

## Impact

The impact is that when those fields are different between chains, one of two things may happen:
1. Less severe - we waste excess gas, which is refunded to the lzReceive() caller (Layer Zero)
2. More severe - we underprice the delivery cost, causing lzReceive() to revert and the NFT stuck in limbo forever.

The code does not handle a failed lzReceive (differently to a failed executeJob). Therefore, no failure event is emitted and the NFT is screwed.

## Tools Used

Manual audit

## Recommended Mitigation Steps

Firstly,make sure to use the target gas costs.
Secondly, re-engineer lzReceive to be fault-proof, i.e. save some gas to emit result event.