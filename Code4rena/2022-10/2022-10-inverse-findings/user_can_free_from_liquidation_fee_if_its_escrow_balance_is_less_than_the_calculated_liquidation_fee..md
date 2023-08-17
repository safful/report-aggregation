## Tags

- bug
- 2 (Med Risk)
- primary issue
- satisfactory
- sponsor confirmed
- selected for report
- M-06

# [User can free from liquidation fee if its escrow balance is less than the calculated liquidation fee.](https://github.com/code-423n4/2022-10-inverse-findings/issues/275) 

# Lines of code

https://github.com/code-423n4/2022-10-inverse/blob/main/src/Market.sol#L605-L610


# Vulnerability details

## Impact
User can free from liquidation fee if its escrow balance less than the calculated liquidation fee.

## Proof of Concept
If the `liquidationFeeBps` is enabled, the `gov` should receive the liquidation fee. But if user's escrow balance is less than the calculated liquidation fee, `gov` got nothing.
https://github.com/code-423n4/2022-10-inverse/blob/main/src/Market.sol#L605-L610

```solidity
        if(liquidationFeeBps > 0) {
            uint liquidationFee = repaidDebt * 1 ether / price * liquidationFeeBps / 10000;
            if(escrow.balance() >= liquidationFee) {
                escrow.pay(gov, liquidationFee);
            }
        }
```


## Tools Used
manual review

## Recommended Mitigation Steps
User should pay all the remaining escrow balance if the calculated liquidation fee is greater than its escrow balance.

```solidity
        if(liquidationFeeBps > 0) {
            uint liquidationFee = repaidDebt * 1 ether / price * liquidationFeeBps / 10000;
            if(escrow.balance() >= liquidationFee) {
                escrow.pay(gov, liquidationFee);
            } else {
                escrow.pay(gov, escrow.balance());
            }
        }
```

