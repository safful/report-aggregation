## Tags

- bug
- 2 (Med Risk)
- resolved
- sponsor confirmed

# [OpenLevV1.closeTrade with V3 DEX doesn't correctly accounts fee on transfer tokens for repayments](https://github.com/code-423n4/2022-01-openleverage-findings/issues/104) 

# Handle

hyh


# Vulnerability details

## Impact

The amount that OpenLevV1 will receive can be less than V3 DEX indicated as a swap result, while it is used as given for position debt repayment accounting.

This way actual funds received can be less than accounted, leaving to system funds deficit, which can be exploited by a malicious user, draining contract funds with multiple open/close with a taxed token.

In the `trade.depositToken != longToken` case when `flashSell` is used this can imply inability to send remainder funds to a user and the failure of the whole closeTrade function, the end result is a freezing of user's funds within the system.

## Proof of Concept

`trade.depositToken != longToken` case, can be wrong repayment accounting, which will lead to a deficit if the received funds are less than DEX returned `closeTradeVars.receiveAmount`.

As a side effect, `doTransferOut` is done without balance check, so the whole position close can revert, leading to inability to close the position and freeze of user's funds this way:

https://github.com/code-423n4/2022-01-openleverage/blob/main/openleverage-contracts/contracts/OpenLevV1.sol#L197-204


I.e. if there is enough funds in the system they will be drained, if there is not enough funds, user's position close will fail.


V3 sell function doesn't check for balance change, using DEX returned amount as is:

https://github.com/code-423n4/2022-01-openleverage/blob/main/openleverage-contracts/contracts/dex/eth/UniV3Dex.sol#L61-70

## Recommended Mitigation Steps

If fee on tranfer tokens are fully in scope, do control all the accounting and amounts to be returned to a user via balance before/after calculations for DEX V3 logic as well.

