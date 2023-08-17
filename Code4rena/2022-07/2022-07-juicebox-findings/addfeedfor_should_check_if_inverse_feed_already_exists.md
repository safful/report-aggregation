## Tags

- bug
- documentation
- 2 (Med Risk)
- sponsor confirmed
- valid

# [addFeedFor should check if inverse feed already exists](https://github.com/code-423n4/2022-07-juicebox-findings/issues/79) 

# Lines of code

https://github.com/jbx-protocol/juice-contracts-v2-code4rena/blob/828bf2f3e719873daa08081cfa0d0a6deaa5ace5/contracts/JBPrices.sol#L109-L122


# Vulnerability details

## Impact
Potentially inconsistent currency conversions

## Proof of Concept
addFeedFor requires that a price feed for the _currency _base doesn't exist when adding a new price feed but doesn't check if the inverse already exists. This means that two different oracles (potentially with different prices) could be used for _currency -> _base vs. _base -> _currency. Different prices would lead to inconsistent between conversion ratios depending on the direction of the conversion

## Tools Used

## Recommended Mitigation Steps
Change L115 to:
if (feedFor[_currency][_base] != IJBPriceFeed(address(0)) || feedFor[_base][_currency] != IJBPriceFeed(address(0))) revert PRICE_FEED_ALREADY_EXISTS()

