## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [ settleAuction may be impossible if locked at a wrong time.](https://github.com/code-423n4/2021-09-defiprotocol-findings/issues/41) 

# Handle

jonah1005


# Vulnerability details

## Impact
The aution contract decides a new `ibRatio` in the function `settleAuction`. [Auction.sol#L89-L91](https://github.com/code-423n4/2021-09-defiProtocol/blob/main/contracts/contracts/Auction.sol#L89-L91)

```solidity
        uint256 a = factory.auctionMultiplier() * basket.ibRatio();
        uint256 b = (bondTimestamp - auctionStart) * BASE / factory.auctionDecrement();
        uint256 newRatio = a - b;
```

In this equation, `a` would not always be greater than `b`. The ` auctionBonder` may lock the token in `bondForRebalance()` at a point that `a-b` would always revert.

The contract should not allow users to lock the token at the point that not gonna succeed. Given the possible (huge) loss of the user may suffer, I consider this is a medium-risk issue.


## Proof of Concept
Here's a web3.py script to trigger this bug.
```python
basket.functions.publishNewIndex([dai.address], [deposit_amount]).transact()

for i in range(4 * 60 * 24):
    w3.provider.make_request('evm_mine', [])
basket.functions.publishNewIndex([dai.address], [deposit_amount]).transact()

print('auction on going', auction.functions.auctionOngoing().call())
for i in range(20000):
    w3.provider.make_request('evm_mine', [])

all_token = basket.functions.balanceOf(user).call()
basket.functions.approve(auction.address, all_token).transact()
auction.functions.bondForRebalance().transact()
# error Log
# {'code': -32603, 'message': 'Error: VM Exception while processing transaction: reverted with panic code 0x11 (Arithmetic operation underflowed or overflowed outside of an unchecked block)'}
auction.functions.settleAuction([], [], [], [], []).transact()
```


## Tools Used
None

## Recommended Mitigation Steps
Recommend to calculate the new irate in `bondForRebalance`. I understand the `auctionBonder` should take the risk to get the profit. However, the contract should protect the user in the first place when this auction is doomed to fail.

