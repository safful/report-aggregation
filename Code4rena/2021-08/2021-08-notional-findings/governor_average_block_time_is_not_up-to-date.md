## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [Governor average block time is not up-to-date](https://github.com/code-423n4/2021-08-notional-findings/issues/73) 

# Handle

cmichel


# Vulnerability details

## Vulnerability Details
The `GovernorAlpha.MIN_VOTING_PERIOD_BLOCKS = 6700` value indicates an average block time of 12.8956s which was correct a year ago, but at the moment a more accurate block time would be 13.2s, see [blocktime](https://etherscan.io/chart/blocktime).

## Recommended Mitigation Steps
Use a `MIN_VOTING_PERIOD_BLOCKS` of `6545`.

