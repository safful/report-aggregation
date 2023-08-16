## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Credit Line acceptance logic can be simplified to avoid SLOAD in some cases](https://github.com/code-423n4/2021-12-sublime-findings/issues/77) 

# Handle

TomFrenchBlockchain


# Vulnerability details

## Impact
gas costs

## Proof of Concept

https://github.com/code-423n4/2021-12-sublime/blob/9df1b7c4247f8631647c7627a8da9bdc16db8b11/contracts/CreditLine/CreditLine.sol#L600-L604

```
(msg.sender == creditLineConstants[_id].borrower && _requestByLender) || (msg.sender == creditLineConstants[_id].lender && !_requestByLender)
```

is equivalent to 

```
_requestByLender ? (msg.sender == creditLineConstants[_id].borrower) : (msg.sender == creditLineConstants[_id].lender)
```
or
```
msg.sender == (_requestByLender ? creditLineConstants[_id].borrower : creditLineConstants[_id].lender)
```

Which avoid loading the borrower address in the case where the borrower made the request.

## Recommended Mitigation Steps

Use simplified logic

