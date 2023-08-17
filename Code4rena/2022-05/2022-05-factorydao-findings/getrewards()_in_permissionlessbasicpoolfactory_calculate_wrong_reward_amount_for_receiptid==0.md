## Tags

- bug
- 2 (Med Risk)
- disagree with severity
- sponsor confirmed

# [getRewards() in PermissionlessBasicPoolFactory calculate wrong reward amount for receiptId==0](https://github.com/code-423n4/2022-05-factorydao-findings/issues/161) 

# Lines of code

https://github.com/code-423n4/2022-05-factorydao/blob/db415804c06143d8af6880bc4cda7222e5463c0e/contracts/PermissionlessBasicPoolFactory.sol#L156-L173


# Vulnerability details

## Impact
In `getRewards()` of `PermissionlessBasicPoolFactory` contract, there is a check to see that receipt is initialized receipt, but the condition used by code will be true for `receiptId` equal `0`. because `receiptId==0` is not initilized for any pool and the value of `pools[poolId].receipts[0].id` will be `0` so the condition `receipt.id == receiptId` will be passed on `getRewards()`. Any function that depends on `getRewards()` to check that if `receptId` has deposited fund, can be fooled. right now this bug has no direct money loss, but this function doesn't work as it suppose too.

## Proof of Concept
This is `getRewards()` code:
```
    function getRewards(uint poolId, uint receiptId) public view returns (uint[] memory) {
        Pool storage pool = pools[poolId];
        Receipt memory receipt = pool.receipts[receiptId];
        require(pool.id == poolId, 'Uninitialized pool');
        require(receipt.id == receiptId, 'Uninitialized receipt');
        uint nowish = block.timestamp;
        if (nowish > pool.endTime) {
            nowish = pool.endTime;
        }

        uint secondsDiff = nowish - receipt.timeDeposited;
        uint[] memory rewardsLocal = new uint[](pool.rewardsWeiPerSecondPerToken.length);
        for (uint i = 0; i < pool.rewardsWeiPerSecondPerToken.length; i++) {
            rewardsLocal[i] = (secondsDiff * pool.rewardsWeiPerSecondPerToken[i] * receipt.amountDepositedWei) / 1e18;
        }
        return rewardsLocal;
    }
```
if the value of `receiptId` set as `0` then even so `receiptId==0` is not initialized but this line:
```
        require(receipt.id == receiptId, 'Uninitialized receipt');
```
will be passed, because, receipts start from number `1` and `pool.receipts[0]` will have zero value for his fields. This is the code in `deposit()` which is responsible for creating receipt objects.
```
        pool.totalDepositsWei += amount;
        pool.numReceipts++;

        Receipt storage receipt = pool.receipts[pool.numReceipts];
        receipt.id = pool.numReceipts;
        receipt.amountDepositedWei = amount;
        receipt.timeDeposited = block.timestamp;
        receipt.owner = msg.sender;
```
as you can see `pool.numReceipts++` and `pool.receipts[pool.numReceipts]` increase `numReceipts` and use it as receipts index. so receipnts will start from index `1`.
This bug will cause that `getRewards(poolId, 0)` return `0` instead of reverting. any function that depend on reverting of `getRewards()` for uninitialized receipts can be excploited by sending `receipntId` as `0`. this function can be inside this contract or other contracts. (`withdraw` use `getRewards` and we will see that we can create `WithdrawalOccurred` event for `receiptsId` as 0)

## Tools Used
VIM

## Recommended Mitigation Steps
If you want to start from index `1` then add this line too to ensure `receipntId` is not `0` too:
```
require(receiptId > 0, 'Uninitialized receipt');
```
or we could check for uninitialized receipnts with `owner` field as non-zero.


