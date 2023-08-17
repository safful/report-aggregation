## Tags

- bug
- 2 (Med Risk)
- resolved
- sponsor confirmed

# [Tokens with `decimals` larger than `18` are not supported](https://github.com/code-423n4/2022-06-connext-findings/issues/204) 

# Lines of code

https://github.com/code-423n4/2022-06-connext/blob/b4532655071566b33c41eac46e75be29b4a381ed/contracts/contracts/core/connext/helpers/ConnextPriceOracle.sol#L99-L115


# Vulnerability details

For tokens with decimals larger than 18, many functions across the codebase will revert due to underflow.

https://github.com/code-423n4/2022-06-connext/blob/b4532655071566b33c41eac46e75be29b4a381ed/contracts/contracts/core/connext/helpers/ConnextPriceOracle.sol#L99-L115

```solidity
function getPriceFromDex(address _tokenAddress) public view returns (uint256) {
    PriceInfo storage priceInfo = priceRecords[_tokenAddress];
    if (priceInfo.active) {
      uint256 rawTokenAmount = IERC20Extended(priceInfo.token).balanceOf(priceInfo.lpToken);
      uint256 tokenDecimalDelta = 18 - uint256(IERC20Extended(priceInfo.token).decimals());
      uint256 tokenAmount = rawTokenAmount.mul(10**tokenDecimalDelta);
      uint256 rawBaseTokenAmount = IERC20Extended(priceInfo.baseToken).balanceOf(priceInfo.lpToken);
      uint256 baseTokenDecimalDelta = 18 - uint256(IERC20Extended(priceInfo.baseToken).decimals());
      uint256 baseTokenAmount = rawBaseTokenAmount.mul(10**baseTokenDecimalDelta);
      uint256 baseTokenPrice = getTokenPrice(priceInfo.baseToken);
      uint256 tokenPrice = baseTokenPrice.mul(baseTokenAmount).div(tokenAmount);

      return tokenPrice;
    } else {
      return 0;
    }
  }
```

https://github.com/code-423n4/2022-06-connext/blob/b4532655071566b33c41eac46e75be29b4a381ed/contracts/contracts/core/connext/facets/StableSwapFacet.sol#L426

```solidity
precisionMultipliers[i] = 10**uint256(SwapUtils.POOL_PRECISION_DECIMALS - decimals[i]);
```

Chainlink feeds' with decimals > 18 are not supported neither:

https://github.com/code-423n4/2022-06-connext/blob/b4532655071566b33c41eac46e75be29b4a381ed/contracts/contracts/core/connext/helpers/ConnextPriceOracle.sol#L122-L140

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

### Recommendation

Consider checking if decimals > 18 and normalize the value by div the decimals difference.

