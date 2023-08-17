## Tags

- bug
- 2 (Med Risk)
- primary issue
- selected for report
- sponsor confirmed
- M-24

# [Chainlink price feed is not sufficiently validated and can return stale price](https://github.com/code-423n4/2022-12-tigris-findings/issues/655) 

# Lines of code

https://github.com/code-423n4/2022-12-tigris/blob/main/contracts/utils/TradingLibrary.sol#L91-L122


# Vulnerability details

## Impact
As mentioned by https://docs.tigris.trade/protocol/oracle, "Prices provided by the oracle network are also compared to Chainlink's public price feeds for additional security. If prices have more than a 2% difference the transaction is reverted." The Chainlink price verification logic in the following `TradingLibrary.verifyPrice` function serves this purpose. However, besides that `IPrice(_chainlinkFeed).latestAnswer()` uses Chainlink's deprecated `latestAnswer` function, this function also does not guarantee that the price returned by the Chainlink price feed is not stale. When `assetChainlinkPriceInt != 0` is `true`, it is still possible that `assetChainlinkPriceInt` is stale in which the Chainlink price verification would compare the off-chain price against a stale price returned by the Chainlink price feed. For a off-chain price that has more than a 2% difference when comparing to a more current price returned by the Chainlink price feed, this off-chain price can be incorrectly considered to have less than a 2% difference when comparing to a stale price returned by the Chainlink price feed. As a result, a trading transaction that should revert can go through, which makes the price verification much less secure.

https://github.com/code-423n4/2022-12-tigris/blob/main/contracts/utils/TradingLibrary.sol#L91-L122
```solidity
    function verifyPrice(
        uint256 _validSignatureTimer,
        uint256 _asset,
        bool _chainlinkEnabled,
        address _chainlinkFeed,
        PriceData calldata _priceData,
        bytes calldata _signature,
        mapping(address => bool) storage _isNode
    )
        external view
    {
        ...
        if (_chainlinkEnabled && _chainlinkFeed != address(0)) {
            int256 assetChainlinkPriceInt = IPrice(_chainlinkFeed).latestAnswer();
            if (assetChainlinkPriceInt != 0) {
                uint256 assetChainlinkPrice = uint256(assetChainlinkPriceInt) * 10**(18 - IPrice(_chainlinkFeed).decimals());
                require(
                    _priceData.price < assetChainlinkPrice+assetChainlinkPrice*2/100 &&
                    _priceData.price > assetChainlinkPrice-assetChainlinkPrice*2/100, "!chainlinkPrice"
                );
            }
        }
    }
```

Based on https://docs.chain.link/docs/historical-price-data, the followings can be done to avoid using a stale price returned by the Chainlink price feed.
1. The `latestRoundData` function can be used instead of the deprecated `latestAnswer` function.
2. `roundId` and `answeredInRound` are also returned. "You can check `answeredInRound` against the current `roundId`. If `answeredInRound` is less than `roundId`, the answer is being carried over. If `answeredInRound` is equal to `roundId`, then the answer is fresh."
3. "A read can revert if the caller is requesting the details of a round that was invalid or has not yet been answered. If you are deriving a round ID without having observed it before, the round might not be complete. To check the round, validate that the timestamp on that round is not 0."

## Proof of Concept
The following steps can occur for the described scenario.
1. Alice calls the `Trading.initiateMarketOrder` function, which eventually calls the `TradingLibrary.verifyPrice` function, to initiate a market order.
2. When the `TradingLibrary.verifyPrice` function is called, the off-chain price is compared to the price returned by the Chainlink price feed for the position asset.
3. The price returned by the Chainlink price feed is stale, and the off-chain price has less than a 2% difference when comparing to this stale price.
4. Alice's `Trading.initiateMarketOrder` transaction goes through. However, this transaction should revert because the off-chain price has more than a 2% difference if comparing to a more current price returned by the Chainlink price feed.

## Tools Used
VSCode

## Recommended Mitigation Steps
https://github.com/code-423n4/2022-12-tigris/blob/main/contracts/utils/TradingLibrary.sol#L113 can be updated to the following code.
```solidity
            (uint80 roundId, int256 assetChainlinkPriceInt, , uint256 updatedAt, uint80 answeredInRound) = IPrice(_chainlinkFeed).latestRoundData();
            require(answeredInRound >= roundId, "price is stale");
            require(updatedAt > 0, "round is incomplete");
```