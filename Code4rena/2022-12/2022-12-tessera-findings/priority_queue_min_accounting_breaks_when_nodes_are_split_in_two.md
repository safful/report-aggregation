## Tags

- bug
- 2 (Med Risk)
- primary issue
- satisfactory
- selected for report
- sponsor confirmed
- M-04

# [Priority queue min accounting breaks when nodes are split in two](https://github.com/code-423n4/2022-12-tessera-findings/issues/32) 

# Lines of code

https://github.com/code-423n4/2022-12-tessera/blob/f37a11407da2af844bbfe868e1422e3665a5f8e4/src/modules/GroupBuy.sol#L299-L319


# Vulnerability details

The README states `If two users place bids at the same price but with different quantities, the queue will pull from the bid with a higher quantity first`, but the data-structure used for implementing this logic, is not used properly and essentially has its data corrupted when a large bid that is the current minimum bid, is split into two parts, so that a more favorable price can be used for a fraction of the large bid. The underlying issue is that one of the tree nodes is modified, without re-shuffling that node's location in the tree.

## Impact
The minimum bid as told by the priority queue will be wrong, leading to the wrong bids being allowed to withdraw their funds, and being kicked out of the fraction of bids that are used to buy the NFT.

## Proof of Concept
The priority queue using a binary tree within an array to [efficiently navigate and find the current minimum based on a node and it children](https://algs4.cs.princeton.edu/24pq/). The sorting of the nodes in the tree is based, in part, on the quantity in the case where two bids have the same price:
```solidity
// File: src/lib/MinPriorityQueue.sol : MinPriorityQueue.isGreater()   #1

111        function isGreater(
112            Queue storage self,
113            uint256 i,
114            uint256 j
115        ) private view returns (bool) {
116            Bid memory bidI = self.bidIdToBidMap[self.bidIdList[i]];
117            Bid memory bidJ = self.bidIdToBidMap[self.bidIdList[j]];
118 @>         if (bidI.price == bidJ.price) {
119 @>             return bidI.quantity <= bidJ.quantity;
120            }
121            return bidI.price > bidJ.price;
122:       }
```
https://github.com/code-423n4/2022-12-tessera/blob/f37a11407da2af844bbfe868e1422e3665a5f8e4/src/lib/MinPriorityQueue.sol#L111-L122

The algorithm of the binary tree only works when the nodes are properly sorted. The sorting is corrupted when a node is modified, without removing it from the tree and re-inserting it:

```solidity
// File: src/modules/GroupBuy.sol : GroupBuy.processBidsInQueue()   #2

299                Bid storage lowestBid = bidPriorityQueues[_poolId].getMin();
300                // Breaks out of while loop if given price is less than than lowest bid price
301                if (_price < lowestBid.price) {
302                    break;
303                }
304    
305                uint256 lowestBidQuantity = lowestBid.quantity;
306                // Checks if lowest bid quantity amount is greater than given quantity amount
307                if (lowestBidQuantity > quantity) {
308                    // Decrements given quantity amount from lowest bid quantity
309 @>                 lowestBid.quantity -= quantity;
310                    // Calculates partial contribution of bid by quantity amount and price
311                    uint256 contribution = quantity * lowestBid.price;
312    
313                    // Decrements partial contribution amount of lowest bid from total and user contributions
314                    totalContributions[_poolId] -= contribution;
315                    userContributions[_poolId][lowestBid.owner] -= contribution;
316                    // Increments pending balance of lowest bid owner
317                    pendingBalances[lowestBid.owner] += contribution;
318    
319:                   // Inserts new bid with given quantity amount into proper position of queue
```
https://github.com/code-423n4/2022-12-tessera/blob/f37a11407da2af844bbfe868e1422e3665a5f8e4/src/modules/GroupBuy.sol#L299-L319

Let's say that the tree looks like this:

```
            A:(p:100,q:10)
            /             \
       B:(p:100,q:10)  C:(<whatever>)
       /           \
D:(whatever)   E:(whatever) 

```

If A is modified so that q (quantity) goes from 10 to 5, B should now be at the root of the tree, since it has the larger size, and would be considered the smaller node. When another node is added, say, `F:(p:100,q:6)`, the algorithm will see that F has a larger size than A, and so A will be popped out as the min, even though B should have been. All nodes that are under B (which may be a lot of the nodes if they all entered at the same price/quantity) essentially become invisible under various scenarios, which means the users that own those bids will not be able to withdraw their funds, even if they really are the lowest bid that deserves to be pushed out of the queue. Note that the swimming up that is done for `F` will not re-shuffle `B` since, according to the algorithm, `F` will start as a child of `C`, and `B` is not in the list of parent nodes of `C`.


## Tools Used
Code inspection

## Recommended Mitigation Steps
When modifying nodes of the tree, remove them first, then re-add them after modification
