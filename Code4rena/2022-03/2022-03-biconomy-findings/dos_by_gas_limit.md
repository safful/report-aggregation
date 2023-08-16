## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [DoS by gas limit](https://github.com/code-423n4/2022-03-biconomy-findings/issues/24) 

# Lines of code

https://github.com/code-423n4/2022-03-biconomy/blob/main/contracts/hyphen/LiquidityFarming.sol#L220
https://github.com/code-423n4/2022-03-biconomy/blob/main/contracts/hyphen/LiquidityFarming.sol#L233


# Vulnerability details

In `deposit` function it is possible to push to `nftIdsStaked` of anyone, an attacker can deposit too many nfts to another user, and when the user will try to withdraw an nft at the end of the list, they will iterate on the list and revert because of gas limit.


