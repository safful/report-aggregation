## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [Unable to remove liquidity in Recovery Mode](https://github.com/code-423n4/2021-11-malt-findings/issues/323) 

# Handle

gzeon


# Vulnerability details

## Impact
According to https://github.com/code-423n4/2021-11-malt#high-level-overview-of-the-malt-protocol
> When the Malt price TWAP drops below a specified threshold (eg 2% below peg) then the protocol will revert any transaction that tries to remove Malt from the AMM pool (ie buying Malt or removing liquidity). Users wanting to remove liquidity can still do so via the UniswapHandler contract that is whitelisted in recovery mode.

However, in https://github.com/code-423n4/2021-11-malt/blob/c3a204a2c0f7c653c6c2dda9f4563fd1dc1cecf3/src/contracts/DexHandlers/UniswapHandler.sol#L236
liquidity removed is directly sent to msg.sender, which would revert if it is not whitelisted
https://github.com/code-423n4/2021-11-malt/blob/c3a204a2c0f7c653c6c2dda9f4563fd1dc1cecf3/src/contracts/PoolTransferVerification.sol#L53

## Recommended Mitigation Steps
Liquidity should be removed to UniswapHandler contract, then the proceed is sent to msg.sender

