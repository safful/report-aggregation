## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [Reentrancy in settleAuction(): malicious publisher can bypass index timelock mechanism, inject malicious index, and rug the basket](https://github.com/code-423n4/2021-09-defiprotocol-findings/issues/31) 

# Handle

kenzo


# Vulnerability details

The settleAuction() function calls withdrawBounty() before setting auctionOngoing = false, thereby allowing reentrancy.

## Impact
A malicious publisher can bypass the index timelock mechanism and publish new index which the basket's users won't have time to respond to.
At worst case, this means setting weights that allow the publisher to withdraw all the basket's underlying funds for himself, under the guise of a valid new index.

## Proof of Concept
1. The publisher (a contract) will propose new valid index and bond the auction.
To settle the auction, the publisher will execute the following steps in the same transaction:
2. Add a bounty of an ERC20 contract with a malicious transfer() function.
3. Settle the valid new weights correctly (using settleAuction() with the correct parameters, and passing the malicious bounty id).
4. settleAuction() will call withdrawBounty() which upon transfer will call the publisher's malicious ERC20 contract.
5. The contract will call settleAuction() again, with empty parameters. Since the previous call's effects have already set all the requirements to be met, settleAuction() will finish correctly and call setNewWeights() which will set the new valid weights and set pendingWeights.pending = false.
6. Still inside the malicious ERC20 contract transfer function, the attacker will now call the basket's publishNewIndex(), with weights that will transfer all the funds to him upon his burning of shares. This call will succeed to set new pending weights as the previous step set pendingWeights.pending = false.
7. Now the malicious withdrawBounty() has ended, and the original settleAuction() is resuming, but now with malicious weights in pendingWeights (set in step 6). settleAuction() will now call setNewWeights() which will set the basket's weights to be the malicious pending weights.
8. Now settleAuction has finished, and the publisher (within the same transaction) will burn() all his shares of the basket, thereby transferring all the tokens to himself.

POC exploit:
Password to both files: "exploit".
AttackPublisher.sol , to be put under contracts/contracts/Exploit: https://pastebin.com/efHZjstS
ExploitPublisher.test.js , to be put under contracts/test: https://pastebin.com/knBtcWkk

## Tools Used
Manual analysis, hardhat.

## Recommended Mitigation Steps
In settleAuction(), move basketAsERC20.transfer() and withdrawBounty() to the end of the function, conforming with Checks Effects Interactions pattern.

