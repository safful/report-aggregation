## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [SafeMath library is not always used in the contracts](https://github.com/code-423n4/2021-11-malt-findings/issues/60) 

# Handle

defsec


# Vulnerability details

## Impact

SafeMath library functions are not always used in arithmetic operations in the contracts, which could potentially cause integer underflow/overflows. Although in the reference lines of code, there are upper limits on the variables to ensure an integer underflow/overflow could not happen, using SafeMath is always a best practice, which prevents underflow/overflows completely (even if there were no assumptions on the variables) and increases code consistency as well.


## Proof of Concept

1. Navigate to the following contracts.

```

https://github.com/code-423n4/2021-11-malt/blob/d3f6a57ba6694b47389b16d9d0a36a956c5e6a94/src/contracts/Auction.sol#L795

https://github.com/code-423n4/2021-11-malt/blob/d3f6a57ba6694b47389b16d9d0a36a956c5e6a94/src/contracts/Auction.sol#L821

https://github.com/code-423n4/2021-11-malt/blob/d3f6a57ba6694b47389b16d9d0a36a956c5e6a94/src/contracts/AuctionBurnReserveSkew.sol#L64

https://github.com/code-423n4/2021-11-malt/blob/d3f6a57ba6694b47389b16d9d0a36a956c5e6a94/src/contracts/AuctionEscapeHatch.sol#L76

https://github.com/code-423n4/2021-11-malt/blob/d3f6a57ba6694b47389b16d9d0a36a956c5e6a94/src/contracts/AuctionEscapeHatch.sol#L188

```

2. SafeMath functions are not used in the every functionality. 

## Tools Used

Code Review

## Recommended Mitigation Steps

Consider using the SafeMath library functions in the referenced lines of code.


