## Tags

- bug
- QA (Quality Assurance)
- sponsor confirmed

# [QA Report](https://github.com/code-423n4/2022-06-connext-findings/issues/190) 

# Chainlink oracle aggregator data is insufficiently validated in `ConnextPriceOracle.sol`

# Lines of code

https://github.com/code-423n4/2022-06-connext/blob/b4532655071566b33c41eac46e75be29b4a381ed/contracts/contracts/core/connext/helpers/ConnextPriceOracle.sol#L122-L140


# Vulnerability details

### Impact

The function `getPriceFromChainlink()` in `ConnextPriceOracle.sol` fetches the `latestRoundData()` from a registered aggregator (Chainlink oracle feed) for a specified token. However, neither round completeness or the quoted timestamp are checked to ensure that the reported price is not stale.

Since Connext is creating bridged representations of tokens from other chains, it is vital for the reported prices of tokens to be accurate. While Connext also consults the DEX reported price of the tokens, some pairs with thin liquidity could also return inaccurate prices.

### Proof of Concept
```solidity
function getPriceFromChainlink(address _tokenAddress) public view returns (uint256) {
    AggregatorV3Interface aggregator = aggregators[_tokenAddress];
    if (address(aggregator) != address(0)) {
      (, int256 answer, , , ) = aggregator.latestRoundData();

      // It's fine for price to be 0. We have two price feeds.
      if (answer == 0) {
        return 0;
      }

      // Extend the decimals to 1e18.
      uint256 retVal = uint256(answer);
      uint256 price = retVal.mul(10**(18 - uint256(aggregator.decimals())));

      return price;
    }

    return 0;
  }
```
Note that the other returned variables `roundId`, `startedAt`, `updatedAt`, and `answeredInRound` are omitted from the return result of `aggregator.latestRoundData()`.

### Mitigation steps
Add additional validation:
```solidity
    ...
     (uint80 roundID, int256 price, , uint256 updatedAt, uint80 answeredInRound) = aggregator.latestRoundData();
    require(answeredInRound >= roundID, "ChainLink: Stale price");
    require(updatedAt != 0, "ChainLink: Round not complete");
    ...
```

### Tools Used
Manual review

