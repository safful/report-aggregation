## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed

# [Timelocked functions doesn't emit proposal events](https://github.com/code-423n4/2021-09-defiprotocol-findings/issues/123) 

# Handle

nikitastupin


# Vulnerability details

Usually timelock is used in order to give a users of a protocol time to react on protocol changes (e.g. to withdraw their funds). Thus timelock implementations have Proposal and Execution steps. The main way to monitor blockchain changes and react to them is to listen for emitted events. However, none of the timelocked functions (`changePublisher`, `changeLicenseFee`, `publishNewIndex`) emits an event on Proposal step (e.g. https://github.com/code-423n4/2021-09-defiProtocol/blob/e6dcf43a2f03aa65e04f0edc8ed1d7272677fabe/contracts/contracts/Basket.sol#L144-L147), they emit an event only on Execution step (e.g. https://github.com/code-423n4/2021-09-defiProtocol/blob/e6dcf43a2f03aa65e04f0edc8ed1d7272677fabe/contracts/contracts/Basket.sol#L143-L143).

## Impact

Events aren't emitted at critical functions.

## Proof of Concept

I'll write a PoC if needed.

## Recommended Mitigation Steps

Add events after (1) https://github.com/code-423n4/2021-09-defiProtocol/blob/e6dcf43a2f03aa65e04f0edc8ed1d7272677fabe/contracts/contracts/Basket.sol#L145-L146, (2) https://github.com/code-423n4/2021-09-defiProtocol/blob/e6dcf43a2f03aa65e04f0edc8ed1d7272677fabe/contracts/contracts/Basket.sol#L163-L164, (3) https://github.com/code-423n4/2021-09-defiProtocol/blob/e6dcf43a2f03aa65e04f0edc8ed1d7272677fabe/contracts/contracts/Basket.sol#L189-L192 and https://github.com/code-423n4/2021-09-defiProtocol/blob/e6dcf43a2f03aa65e04f0edc8ed1d7272677fabe/contracts/contracts/Basket.sol#L182-L186.

