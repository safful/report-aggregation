## Tags

- bug
- 2 (Med Risk)
- disagree with severity
- resolved
- sponsor confirmed

# [Oracle data feed is insufficiently validated.](https://github.com/code-423n4/2022-04-jpegd-findings/issues/54) 

# Lines of code

https://github.com/code-423n4/2022-04-jpegd/blob/e72861a9ccb707ced9015166fbded5c97c6991b6/contracts/vaults/FungibleAssetVaultForDAO.sol#L105


# Vulnerability details

## Impact
Price can be stale and can lead to wrong `answer` return value.

## Proof of Concept
Oracle data feed is insufficiently validated. There is no check for stale price and round completeness. Price can be stale and can lead to wrong  `answer`  return value.

```
function _collateralPriceUsd() internal view returns (uint256) {
        int256 answer = oracle.latestAnswer();
        uint8 decimals = oracle.decimals();

        require(answer > 0, "invalid_oracle_answer");

        ...

```

https://github.com/code-423n4/2022-04-jpegd/blob/e72861a9ccb707ced9015166fbded5c97c6991b6/contracts/vaults/FungibleAssetVaultForDAO.sol#L105

## Tools Used
Manual review
## Recommended Mitigation Steps
Validate data feed

```
function _collateralPriceUsd() internal view returns (uint256) {

(uint80 roundID, int256 answer, , uint256 timestamp, uint80 answeredInRound) = oracle.latestRoundData();
   
    require(answer > 0, "invalid_oracle_answer");
    require(answeredInRound >= roundID, "ChainLink: Stale price");
    require(timestamp > 0, "ChainLink: Round not complete");

         ...

```

