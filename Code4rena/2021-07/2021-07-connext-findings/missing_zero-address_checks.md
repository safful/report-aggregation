## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [Missing zero-address checks](https://github.com/code-423n4/2021-07-connext-findings/issues/50) 

# Handle

0xRajeev


# Vulnerability details

## Impact

Zero-address checks are in general a best-practice. However, addLiquidity() and removeLiquidity() are missing zero-address checks on router and recipient addresses respectively.  

addLiquidity() on Eth transfers will update the zero index balance and get logged as such in the event without the amount getting accounted for the correct router.

For ERC20 assets, token.transfer() generally implements this check but the Eth transfer using transferEth() does not have this check and calls addr.call(value) which will lead to burning in the case of removeLiquidity(). 

The checks may be more important because assetID is 0 for Eth. So a router may accidentally use 0 values for both assetID and router/recipient.

There is also a missing zero-address check on sendingChainFallback which is relevant for Eth transfers in cancel(). The comment on L178 indicates the need for this but the following check on L179 ends up checking receivingAddress instead (which is also necessary).


## Proof of Concept

https://github.com/code-423n4/2021-07-connext/blob/8e1a7ea396d508ed2ebeba4d1898a748255a48d2/contracts/TransactionManager.sol#L88
https://github.com/code-423n4/2021-07-connext/blob/8e1a7ea396d508ed2ebeba4d1898a748255a48d2/contracts/TransactionManager.sol#L101-L104

https://github.com/code-423n4/2021-07-connext/blob/8e1a7ea396d508ed2ebeba4d1898a748255a48d2/contracts/TransactionManager.sol#L116
https://github.com/code-423n4/2021-07-connext/blob/8e1a7ea396d508ed2ebeba4d1898a748255a48d2/contracts/TransactionManager.sol#L128-L131

https://github.com/code-423n4/2021-07-connext/blob/8e1a7ea396d508ed2ebeba4d1898a748255a48d2/contracts/TransactionManager.sol#L504


## Tools Used

Manual Analysis

## Recommended Mitigation Steps

Add zero-address checks.

