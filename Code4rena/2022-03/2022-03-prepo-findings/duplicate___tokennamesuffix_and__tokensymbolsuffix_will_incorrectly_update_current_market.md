## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Duplicate  _tokenNameSuffix and _tokenSymbolSuffix will incorrectly update current Market](https://github.com/code-423n4/2022-03-prepo-findings/issues/2) 

# Lines of code

https://github.com/code-423n4/2022-03-prepo/blob/main/contracts/core/PrePOMarketFactory.sol#L42


# Vulnerability details

## Impacted Function:
createMarket

## Description:

1. Owner calls createMarket with  _tokenNameSuffix S1 and _tokenSymbolSuffix S2 which creates a new market M1 with _deployedMarkets[_salt] pointing to M1. Here salt can be S which is computed using  _tokenNameSuffix and _tokenSymbolSuffix
2. This market is now being used
3. After some time owner again mistakenly calls createMarket with  _tokenNameSuffix S1 and _tokenSymbolSuffix S2 
4. Instead of returning error mentioning that this name and symbol already exists, new market gets created. The problem here is that salt which is computed using _tokenNameSuffix and _tokenSymbolSuffix will again come as S (as in step 1) which means _deployedMarkets[_salt] will now get updated to M2. This means reference to M1 is gone

## Recommendation:
Add below check:

```
require(_deployedMarkets[_salt]==address(0), "Market already exists");
```

