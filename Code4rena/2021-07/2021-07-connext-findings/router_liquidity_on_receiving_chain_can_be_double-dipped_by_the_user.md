## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [Router liquidity on receiving chain can be double-dipped by the user](https://github.com/code-423n4/2021-07-connext-findings/issues/46) 

# Handle

0xRajeev


# Vulnerability details

## Impact

During fulfill() on the receiving chain, if the user has set up an external contract at txData.callTo, the catch blocks for both IFulfillHelper.addFunds() and IFulfillHelper.excute() perform transferAsset to the predetermined fallback address txData.receivingAddress.

If addFunds() has reverted earlier, toSend amount would already have been transferred to the receivingAddress. If execute() also fails, it is again transferred. 

Scenario: User sets up receiver chain txData.callTo contract such that both addFunds() and execute() calls revert and that will let him get twice the toSend amount credited to the receivingAddress. So effectively, Alice locks 100 tokenAs on chain A and can get 200 tokenAs (or twice the amount of any token she is supposed to get on chainB from the router), minus relayer fee, on chainB. Router liquidity is double-dipped by Alice and router loses funds.

## Proof of Concept

https://github.com/code-423n4/2021-07-connext/blob/8e1a7ea396d508ed2ebeba4d1898a748255a48d2/contracts/TransactionManager.sol#L395-L409

https://github.com/code-423n4/2021-07-connext/blob/8e1a7ea396d508ed2ebeba4d1898a748255a48d2/contracts/TransactionManager.sol#L413-L428

## Tools Used

Manual Analysis

## Recommended Mitigation Steps

The second catch block for execute() should likely not have the transferAsset() call. It seems like a copy-and-paste bug unless there is some reason that is outside the specified scope and documentation for this contest.

