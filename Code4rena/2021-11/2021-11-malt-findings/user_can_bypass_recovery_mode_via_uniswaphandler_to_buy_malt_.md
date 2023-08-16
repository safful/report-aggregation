## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [User can bypass Recovery Mode via UniswapHandler to buy Malt ](https://github.com/code-423n4/2021-11-malt-findings/issues/325) 

# Handle

gzeon


# Vulnerability details

## Impact
One of the innovative feature of Malt is to block buying while under peg. The buy block can be bypassed by swapping to the whitelisted UniswapHandler, and then extract the token by abusing the add and remove liquidity function. This is considered a high severity issue because it undermine to protocol's ability to generate profit by the privileged role as designed and allow potential risk-free MEV.

## Proof of Concept
1) User swap dai into malt and send malt directly to uniswapHandler, this is possible becuase uniswapHandler is whitelisted
`swapExactTokensForTokens(amountDai, 0, [dai.address, malt.address], uniswapHandler.address, new Date().getTime() + 10000);`
2) User send matching amount of dai to uniswapHandler
3) User call addLiquidity() and get back LP token
4) User call removeLiquidity() and get back both dai and malt

## Recommended Mitigation Steps
According to documentation in https://github.com/code-423n4/2021-11-malt#high-level-overview-of-the-malt-protocol
> Users wanting to remove liquidity can still do so via the UniswapHandler contract that is whitelisted in recovery mode.
, this should be exploitable. Meanwhile the current implementation did not actually allow remove liquidity during recovery mode (refer to issue "Unable to remove liquidity in Recovery Mode")
This exploit can be mitigated by disabling addLiquidity() when the protocol is in recovery mode

