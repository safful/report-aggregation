## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Inconsistency in staleness checks between OHM and reserve token oracles](https://github.com/code-423n4/2022-08-olympus-findings/issues/391) 

# Lines of code

https://github.com/code-423n4/2022-08-olympus/blob/main/src/modules/PRICE.sol#L165-L171


# Vulnerability details

## Impact
Price oracle may fail and revert due to the inconsistency in the staleness checks. 

## Proof of Concept

In the `getCurrentPrice()` of `PRICE.sol`, Chainlink oracles are used to get the price of OHM against a reserve token, and a staleness check is used to make sure the price oracles are reporting fresh data. Yet the freshness requirements are inconsistent, for OHM, `updatedAt` should be lower than current timestamp minus three times the observation frequency, while for the reserve price, it is required that `updatedAt` should be lower than current timestamp minus the observation frequency. Our understanding is that that frequency is multiplied by 3 so that there can be some meaningful room where price data is accepted, as the time frame of only observation frequency (multiplied by 1) may not be enough for the oracle to realistically update its data.  (In other words, the frequency of new price information might be lower than the observation frequency, which is probably the case as third multiple is used for the OHM price).  If this is the case, this inconsistency may lead to the `getCurrentPrice()` reverting as while third multiple of the observation frequency might give enough space for the first oracle, second oracle's first multiple of frequency time frame might not be enough and it couldn't pass the staleness check due to unrealistic expectation of freshness. 

## Tools Used
Manual review, talking with devs

## Recommended Mitigation Steps
Change the line 171 to 
```
            if (updatedAt < block.timestamp - 3 * uint256(observationFrequency))
```
like line 165. 