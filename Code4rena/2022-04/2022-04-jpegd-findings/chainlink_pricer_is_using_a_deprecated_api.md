## Tags

- bug
- 2 (Med Risk)
- resolved
- sponsor confirmed

# [Chainlink pricer is using a deprecated API](https://github.com/code-423n4/2022-04-jpegd-findings/issues/4) 

# Lines of code

https://github.com/code-423n4/2022-04-jpegd/blob/main/contracts/vaults/FungibleAssetVaultForDAO.sol#L105


# Vulnerability details

## Impact
According to Chainlink's documentation, the latestAnswer function is deprecated. This function might suddenly stop working if Chainlink stop supporting deprecated APIs. And the old API can return stale data.

## Proof of Concept
https://github.com/code-423n4/2022-04-jpegd/blob/main/contracts/vaults/FungibleAssetVaultForDAO.sol#L105
https://github.com/code-423n4/2022-04-jpegd/blob/main/contracts/vaults/NFTVault.sol#L459
## Tools Used
None
## Recommended Mitigation Steps
Use the latestRoundData function to get the price instead. Add checks on the return data with proper revert messages if the price is stale or the round is uncomplete
https://docs.chain.link/docs/price-feeds-api-reference/

