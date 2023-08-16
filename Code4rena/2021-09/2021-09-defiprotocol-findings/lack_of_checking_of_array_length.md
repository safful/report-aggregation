## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [lack of checking of array length](https://github.com/code-423n4/2021-09-defiprotocol-findings/issues/111) 

# Handle

JMukesh


# Vulnerability details

## Impact
due to lack of checking of array parameters in settleAuction()  , these array parameters can have different length which can lead to error.  inputWeight is iterated over the length of inputToken if one of the parameter have less length than other one will become inaccessible   which can lead to error

## Proof of Concept

https://github.com/code-423n4/2021-09-defiProtocol/blob/52b74824c42acbcd64248f68c40128fe3a82caf6/contracts/contracts/Auction.sol#L69

## Tools Used

manual review

## Recommended Mitigation Steps

