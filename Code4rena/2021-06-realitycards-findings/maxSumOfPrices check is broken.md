## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed
- resolved

# [maxSumOfPrices check is broken](https://github.com/code-423n4/2021-06-realitycards-findings/issues/87) 

# Handle

0xRajeev


# Vulnerability details

## Impact

rentAllCards() requires the sender to specify a _maxSumOfPrices parameter which specifies “limit to the sum of the bids to place” as specified in the Natspec @param comment. This is apparently for front-run protection.

However, this function parameter constraint for _maxSumOfPrices is broken in the function implementation which leads to the total of bids places greater than the _maxSumOfPrices specified.

Impact: The user may not have sufficient deposit, be foreclosed and/or impacted on other bids/markets.

## Proof of Concept

Scenario: Assume two cards for a market with current winning rentals of 50 each. _maxSumofPrices = 101 passes check on L643 but then the forced 10% increase on L650 (assuming sender is not the owner of either card) causes newRentals to be called with 55 for each card thus totalling to 110 which is > 101 as requested by the user.

https://github.com/code-423n4/2021-06-realitycards/blob/86a816abb058cc0ed9b6f5c4a8ad146f22b8034c/contracts/RCMarket.sol#L636-L637

https://github.com/code-423n4/2021-06-realitycards/blob/86a816abb058cc0ed9b6f5c4a8ad146f22b8034c/contracts/RCMarket.sol#L639-L657


## Tools Used

Manual Analysis

## Recommended Mitigation Steps

Modify the max sum of prices check logic to consider the 10% increase scenarios. Document and suggest the max sum of prices for the user in the UI based on the card prices and 10% requirement depending on card ownership.

