## Tags

- bug
- 2 (Med Risk)
- primary issue
- sponsor confirmed
- selected for report
- M-17

# [Chainlink oracle data feed is not sufficiently validated and can return stale `price`](https://github.com/code-423n4/2022-10-inverse-findings/issues/584) 

# Lines of code

https://github.com/code-423n4/2022-10-inverse/blob/main/src/Oracle.sol#L78-L105
https://github.com/code-423n4/2022-10-inverse/blob/main/src/Oracle.sol#L112-L144
https://github.com/code-423n4/2022-10-inverse/blob/main/src/Market.sol#L344-L347
https://github.com/code-423n4/2022-10-inverse/blob/main/src/Market.sol#L323-L327
https://github.com/code-423n4/2022-10-inverse/blob/main/src/Market.sol#L353-L363


# Vulnerability details

## Impact
Calling the `Oracle` contract's `viewPrice` or `getPrice` function executes `uint price = feeds[token].feed.latestAnswer()` and `require(price > 0, "Invalid feed price")`. Besides that Chainlink's `latestAnswer` function is deprecated, only verifying that `price > 0` is true is also not enough to guarantee that the returned `price` is not stale. Using a stale `price` can cause the calculations for the credit and withdrawal limits to be inaccurate, which, for example, can mistakenly consider a user's debt to be under water and unexpectedly allow the user's debt to be liquidated.

To avoid using a stale answer returned by the Chainlink oracle data feed, according to [Chainlink's documentation](https://docs.chain.link/docs/historical-price-data):
1. The `latestRoundData` function can be used instead of the deprecated `latestAnswer` function.
2. `roundId` and `answeredInRound` are also returned. "You can check `answeredInRound` against the current `roundId`. If `answeredInRound` is less than `roundId`, the answer is being carried over. If `answeredInRound` is equal to `roundId`, then the answer is fresh."
3. "A read can revert if the caller is requesting the details of a round that was invalid or has not yet been answered. If you are deriving a round ID without having observed it before, the round might not be complete. To check the round, validate that the timestamp on that round is not 0."



https://github.com/code-423n4/2022-10-inverse/blob/main/src/Oracle.sol#L78-L105
```solidity
    function viewPrice(address token, uint collateralFactorBps) external view returns (uint) {
        if(fixedPrices[token] > 0) return fixedPrices[token];
        if(feeds[token].feed != IChainlinkFeed(address(0))) {
            // get price from feed
            uint price = feeds[token].feed.latestAnswer();
            require(price > 0, "Invalid feed price");
            // normalize price
            uint8 feedDecimals = feeds[token].feed.decimals();
            uint8 tokenDecimals = feeds[token].tokenDecimals;
            uint8 decimals = 36 - feedDecimals - tokenDecimals;
            uint normalizedPrice = price * (10 ** decimals);
            uint day = block.timestamp / 1 days;
            // get today's low
            uint todaysLow = dailyLows[token][day];
            // get yesterday's low
            uint yesterdaysLow = dailyLows[token][day - 1];
            // calculate new borrowing power based on collateral factor
            uint newBorrowingPower = normalizedPrice * collateralFactorBps / 10000;
            uint twoDayLow = todaysLow > yesterdaysLow && yesterdaysLow > 0 ? yesterdaysLow : todaysLow;
            if(twoDayLow > 0 && newBorrowingPower > twoDayLow) {
                uint dampenedPrice = twoDayLow * 10000 / collateralFactorBps;
                return dampenedPrice < normalizedPrice ? dampenedPrice: normalizedPrice;
            }
            return normalizedPrice;

        }
        revert("Price not found");
    }
```

https://github.com/code-423n4/2022-10-inverse/blob/main/src/Oracle.sol#L112-L144
```solidity
    function getPrice(address token, uint collateralFactorBps) external returns (uint) {
        if(fixedPrices[token] > 0) return fixedPrices[token];
        if(feeds[token].feed != IChainlinkFeed(address(0))) {
            // get price from feed
            uint price = feeds[token].feed.latestAnswer();
            require(price > 0, "Invalid feed price");
            // normalize price
            uint8 feedDecimals = feeds[token].feed.decimals();
            uint8 tokenDecimals = feeds[token].tokenDecimals;
            uint8 decimals = 36 - feedDecimals - tokenDecimals;
            uint normalizedPrice = price * (10 ** decimals);
            // potentially store price as today's low
            uint day = block.timestamp / 1 days;
            uint todaysLow = dailyLows[token][day];
            if(todaysLow == 0 || normalizedPrice < todaysLow) {
                dailyLows[token][day] = normalizedPrice;
                todaysLow = normalizedPrice;
                emit RecordDailyLow(token, normalizedPrice);
            }
            // get yesterday's low
            uint yesterdaysLow = dailyLows[token][day - 1];
            // calculate new borrowing power based on collateral factor
            uint newBorrowingPower = normalizedPrice * collateralFactorBps / 10000;
            uint twoDayLow = todaysLow > yesterdaysLow && yesterdaysLow > 0 ? yesterdaysLow : todaysLow;
            if(twoDayLow > 0 && newBorrowingPower > twoDayLow) {
                uint dampenedPrice = twoDayLow * 10000 / collateralFactorBps;
                return dampenedPrice < normalizedPrice ? dampenedPrice: normalizedPrice;
            }
            return normalizedPrice;

        }
        revert("Price not found");
    }
```

https://github.com/code-423n4/2022-10-inverse/blob/main/src/Market.sol#L344-L347
```solidity
    function getCreditLimitInternal(address user) internal returns (uint) {
        uint collateralValue = getCollateralValueInternal(user);
        return collateralValue * collateralFactorBps / 10000;
    }
```

https://github.com/code-423n4/2022-10-inverse/blob/main/src/Market.sol#L323-L327
```solidity
    function getCollateralValueInternal(address user) internal returns (uint) {
        IEscrow escrow = predictEscrow(user);
        uint collateralBalance = escrow.balance();
        return collateralBalance * oracle.getPrice(address(collateral), collateralFactorBps) / 1 ether;
    }
```

https://github.com/code-423n4/2022-10-inverse/blob/main/src/Market.sol#L353-L363
```solidity
    function getWithdrawalLimitInternal(address user) internal returns (uint) {
        IEscrow escrow = predictEscrow(user);
        uint collateralBalance = escrow.balance();
        if(collateralBalance == 0) return 0;
        uint debt = debts[user];
        if(debt == 0) return collateralBalance;
        if(collateralFactorBps == 0) return 0;
        uint minimumCollateral = debt * 1 ether / oracle.getPrice(address(collateral), collateralFactorBps) * 10000 / collateralFactorBps;
        if(collateralBalance <= minimumCollateral) return 0;
        return collateralBalance - minimumCollateral;
    }
```

## Proof of Concept
The following steps can occur for the described scenario.
1. Alice calls the `depositAndBorrow` function to deposit some WETH as the collateral and borrows some DOLA against the collateral.
2. Bob calls the `liquidate` function for trying to liquidate Alice's debt. Because the Chainlink oracle data feed returns an up-to-date price at this moment, the `getCreditLimitInternal` function calculates Alice's credit limit accurately, which does not cause Alice's debt to be under water. Hence, Bob's `liquidate` transaction reverts.
3. After some time, Bob calls the `liquidate` function again for trying to liquidate Alice's debt. This time, because the Chainlink oracle data feed returns a positive but stale price, the `getCreditLimitInternal` function calculates Alice's credit limit inaccurately, which mistakenly causes Alice's debt to be under water.
4. Bob's `liquidate` transaction is executed successfully so he gains some of Alice's WETH collateral. Alice loses such WETH collateral amount unexpectedly because her debt should not be considered as under water if the stale price was not used.

## Tools Used
VSCode

## Recommended Mitigation Steps
https://github.com/code-423n4/2022-10-inverse/blob/main/src/Oracle.sol#L82-L83 and https://github.com/code-423n4/2022-10-inverse/blob/main/src/Oracle.sol#L116-L117 can be updated to the following code.
```solidity
            (uint80 roundId, int256 answer, , uint256 updatedAt, uint80 answeredInRound) = feeds[token].feed.latestRoundData();
            require(answeredInRound >= roundId, "answer is stale");
            require(updatedAt > 0, "round is incomplete");
            require(answer > 0, "Invalid feed answer");

            uint256 price = uint256(answer);
```