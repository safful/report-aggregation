## Tags

- bug
- 2 (Med Risk)
- resolved
- sponsor confirmed
- selected for report

# [Different Oracle issues can return outdated prices](https://github.com/code-423n4/2022-09-y2k-finance-findings/issues/61) 

# Lines of code

https://github.com/code-423n4/2022-09-y2k-finance/blob/ac3e86f07bc2f1f51148d2265cc897e8b494adf7/src/oracles/PegOracle.sol#L63
https://github.com/code-423n4/2022-09-y2k-finance/blob/ac3e86f07bc2f1f51148d2265cc897e8b494adf7/src/Controller.sol#L308
https://github.com/code-423n4/2022-09-y2k-finance/blob/ac3e86f07bc2f1f51148d2265cc897e8b494adf7/src/oracles/PegOracle.sol#L126


# Vulnerability details

## Impact
Different problems have been found with the use of the oracle that can incur economic losses when the oracle is not consumed in a completely safe way.

## Proof of Concept

Thre problems found are:

- The `timeStamp` check is not correct since in both cases it is done against 0, which would mean that a date of 2 years ago would be valid, so old prices can be taken.

```javascript
    function getLatestPrice(address _token)
        public
        view
        returns (int256 nowPrice)
    {
        ...
        if(timeStamp == 0)
            revert TimestampZero();
        return price;
    }
```

- Oracle price 1 can be outdated:

The `latestRoundData` method of the `PegOracle` contract calls `priceFeed1.latestRoundData();` directly, but does not perform the necessary round or timestamp checks, and delegates them to the caller, but these checks are performed on price2 because it calls `getOracle2_Price` in this case, this inconsistency between how it take the price1 and price2 behaves favors human errors when consuming the oracle.

## Recommended Mitigation Steps

For the timestamp issue, it should be checked like this:

```diff
+   uint constant observationFrequency = 1 hours;

    function getLatestPrice(address _token)
        public
        view
        returns (int256 nowPrice)
    {
        ...
        (
            uint80 roundID,
            int256 price,
            ,
            uint256 timeStamp,
            uint80 answeredInRound
        ) = priceFeed.latestRoundData();

        uint256 decimals = 10**(18-(priceFeed.decimals()));
        price = price * int256(decimals);

        if(price <= 0)
            revert OraclePriceZero();

        if(answeredInRound < roundID)
            revert RoundIDOutdated();

-       if(timeStamp == 0)
+       if(timeStamp < block.timestamp - uint256(observationFrequency))
            revert TimestampZero();

        return price;
    }
```


