## Tags

- bug
- 1 (Low Risk)
- resolved
- sponsor confirmed

# [Skip balance check in _beforeTokenTransfer if no withdrawalRequest exists](https://github.com/code-423n4/2022-01-insure-findings/issues/31) 

# Handle

TomFrenchBlockchain


# Vulnerability details

## Impact

Gas costs

## Proof of Concept

When transferring any of the market tokens, a check is performed to see if they have a pending withdrawal and reduce it if their balance falls below the requested amount.

https://github.com/code-423n4/2022-01-insure/blob/19d1a7819fe7ce795e6d4814e7ddf8b8e1323df3/contracts/PoolTemplate.sol#L910-L923

In the case where a user has no pending withdrawal we then perform an unnecessary check on their balance. We could save an SLOAD by changing it to the below

```
if (from != address(0)) {
    uint256 reqAmount = withdrawalReq[from].amount
    if (reqAmount > 0){
        uint256 _after = balanceOf(from) - amount;
        if (_after < reqAmount) {
            withdrawalReq[from].amount = _after;
        }
    } 
}
```

## Recommended Mitigation Steps

As above

