## Tags

- bug
- 3 (High Risk)
- high quality report
- resolved
- sponsor confirmed
- edited-by-warden
- selected for report

# [Incorrect handling of pricefeed.decimals()](https://github.com/code-423n4/2022-09-y2k-finance-findings/issues/195) 

# Lines of code

https://github.com/code-423n4/2022-09-y2k-finance/blob/2175c044af98509261e4147edeb48e1036773771/src/oracles/PegOracle.sol#L46-L83
https://github.com/code-423n4/2022-09-y2k-finance/blob/2175c044af98509261e4147edeb48e1036773771/src/Controller.sol#L299-L300


# Vulnerability details

## Impact
Wrong maths for handling pricefeed decimals. This code will only work for pricefeeds of 8 decimals, any others give wrong/incorrect data. The maths used can be shown in three lines:

```solidity
nowPrice = (price1 * 10000) / price2;
nowPrice = nowPrice * int256(10**(18 - priceFeed1.decimals()));
return nowPrice / 1000000;
```

Line1: adds 4 decimals
Line2: adds (18 - d) decimals, (where d = pricefeed.decimals())
Line3:  removes 6 decimals

Total: adds (16 - d) decimals

when d=8, the contract correctly returns an 8 decimal number. However, when d = 6, the function will return a 10 decimal number. This is further raised by (18-d = 12) decimals when checking for depeg event, leading to a 22 decimal number which is 4 orders of magnitude incorrect.

if d=18, (like usd-eth pricefeeds) contract fails / returns 0.

All chainlink contracts which give price in eth, operate with 18 decimals. So this can cripple the system if added later.

## Proof of Concept
Running the test  AssertTest.t.sol:testPegOracleMarketCreation and changing the line on

https://github.com/code-423n4/2022-09-y2k-finance/blob/2175c044af98509261e4147edeb48e1036773771/test/AssertTest.t.sol#L30

to
```solidity
PegOracle pegOracle3 = new PegOracle(
            0xB1552C5e96B312d0Bf8b554186F846C40614a540,  //usd-eth contract address
            btcEthOracle
        );
```
gives this output

```
oracle3price1: 1085903802394919427                                                                                                                                                                               
oracle3price2: 13753840915281064000                                                                                                                                                                              
oracle3price1 / oracle3price2: 0
```

returning an oracle value of 0. Simulating with a mock price feed of 6 decimals gives results 4 orders of magnitude off.

## Tools Used
Foundry, vs-code

## Recommended Mitigation Steps
Since only the price ratio is calculated, there is no point in increasing the decimal by (18-d) in the second line. Proposed solution:
```solidity
nowPrice = (price1 * 10000) / price2;
nowPrice = nowPrice * int256(10**(priceFeed1.decimals())) * 100;
return nowPrice / 1000000;
```
This returns results in d decimals, no matter the value of d.
