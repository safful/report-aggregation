## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [USDM locked unless guardian remove liquidity](https://github.com/code-423n4/2022-02-concur-findings/issues/187) 

# Lines of code

https://github.com/code-423n4/2022-02-concur/blob/72b5216bfeaa7c52983060ebfc56e72e0aa8e3b0/contracts/USDMPegRecovery.sol#L90


# Vulnerability details

## Impact
In README.me:
> USDM deposits are locked based on the KPI’s from carrot.eth

However, USDM deposits are also locked until guardian remove liquidity because there are no mechanism to remove deposited USDM in `withdraw`

https://github.com/code-423n4/2022-02-concur/blob/72b5216bfeaa7c52983060ebfc56e72e0aa8e3b0/contracts/USDMPegRecovery.sol#L90

