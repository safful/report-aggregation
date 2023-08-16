## Tags

- bug
- 2 (Med Risk)
- disagree with severity
- sponsor confirmed

# [`Oracle.getUnderlyingPrice` could have wrong decimals](https://github.com/code-423n4/2022-02-hubble-findings/issues/44) 

# Lines of code

https://github.com/code-423n4/2022-02-hubble/blob/8c157f519bc32e552f8cc832ecc75dc381faa91e/contracts/Oracle.sol#L34


# Vulnerability details

## Impact
The `Oracle.getUnderlyingPrice` function divides the chainlink price by `100`.
It probably assumes that the answer for the underlying is in 8 decimals but then wants to reduce it for 6 decimals to match USDC.

However, arbitrary `underlying` tokens are used and the chainlink oracles can have different decimals.

## Recommended Mitigation Steps
While most USD price feeds use 8 decimals, it's better to take the on-chain reported decimals into account by doing `AggregatorV3Interface(chainLinkAggregatorMap[underlying]).decimals()`, see [Chainlink docs](https://docs.chain.link/docs/get-the-latest-price/#getting-a-different-price-denomination).
The price should then be scaled down to 6 decimals.


