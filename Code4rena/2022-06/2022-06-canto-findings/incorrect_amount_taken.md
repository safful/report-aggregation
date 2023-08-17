## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Incorrect amount taken](https://github.com/code-423n4/2022-06-canto-findings/issues/98) 

# Lines of code

https://github.com/Plex-Engineer/lending-market/blob/755424c1f9ab3f9f0408443e6606f94e4f08a990/contracts/CNote.sol#L129


# Vulnerability details

## Impact
It was observed that in repayBorrowFresh function, User is asked to send repayAmount instead of repayAmountFinal. This can lead to loss of user funds as user might be paying extra

## Proof of Concept
1. User is making a repayment which eventually calls repayBorrowFresh function

2. Assuming repayAmount == type(uint).max, so repayAmountFinal becomes accountBorrowsPrev

3. This means User should only transfer in accountBorrowsPrev instead of repayAmount but that is not true. Contract is transferring repayAmount instead of repayAmountFinal as seen at CNote.sol#L129

```
uint actualRepayAmount = doTransferIn(payer, repayAmount);
```

## Recommended Mitigation Steps
Revise CNote.sol#L129 to below:

```
uint actualRepayAmount = doTransferIn(payer, repayAmountFinal);
```

