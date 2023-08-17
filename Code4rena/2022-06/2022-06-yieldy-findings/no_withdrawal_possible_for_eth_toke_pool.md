## Tags

- bug
- 3 (High Risk)
- resolved
- sponsor confirmed

# [No withdrawal possible for ETH TOKE pool](https://github.com/code-423n4/2022-06-yieldy-findings/issues/87) 

# Lines of code

https://github.com/code-423n4/2022-06-yieldy/blob/34774d3f5e9275978621fd20af4fe466d195a88b/src/contracts/Staking.sol#L308


# Vulnerability details

## Impact
The `withdraw` function of the ETH Tokemak pool has an additional parameter `asEth`. This can be seen in the Tokemak [Github repository](https://github.com/Tokemak/tokemak-smart-contracts-public/blob/2f54689d5d16ddfd1751493b161a049d6c98c382/contracts/pools/EthPool.sol#L94) or also when looking at the deployed code of the [ETH pool](https://etherscan.io/address/0xb104A7fA1041168556218DDb40Fe2516F88246d5#code). Compare that to e.g. the [USDC pool](https://etherscan.io/address/0xca5e07804beef19b6e71b9db18327d215cd58d4e#code), which does not have this parameter.

This means that the call to `withdraw` will when the staking token is ETH / WETH and no withdrawals would be possible.

## Proof of Concept
A new `Staking` contract with ETH / WETH as the staking token is deployed. Deposits in Tokemak work fine, so users stake their tokens. However, because of the previously described issue, no withdrawal is possible, leaving the funds locked.

## Recommended Mitigation Steps
Handle the case where the underlying asset is WETH / ETH separately and pass this boolean in that case.

