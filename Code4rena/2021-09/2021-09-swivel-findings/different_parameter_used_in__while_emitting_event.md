## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed
- disagree with severity

# [Different parameter used in  while emitting event](https://github.com/code-423n4/2021-09-swivel-findings/issues/90) 

# Handle

JMukesh


# Vulnerability details

## Impact
In matureMarket(address u, uint256 m) function , different parameter is used in event

    emit Mature(u, m, block.timestamp, currentExchangeRate);

maturity rate should before matured timestamp
//   event Mature(address indexed underlying, uint256 indexed maturity, uint256 maturityRate, uint256 matured);

impact of this can be severe .   This   error   may   negatively   impact   
off-chain   tools   that   are   monitoring   events data

## Proof of Concept
https://github.com/Swivel-Finance/gost/blob/5fb7ad62f1f3a962c7bf5348560fe88de0618bae/test/marketplace/MarketPlace.sol#L88

## Tools Used
manual review

## Recommended Mitigation Steps


