## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Changing `bribeVault` in `RewardDistributor.sol` will Lock Current ETH Rewards](https://github.com/code-423n4/2022-02-redacted-cartel-findings/issues/7) 

# Lines of code

https://github.com/code-423n4/2022-02-redacted-cartel/blob/main/contracts/RewardDistributor.sol#L178-#L182
https://github.com/code-423n4/2022-02-redacted-cartel/blob/main/contracts/RewardDistributor.sol#L65-#L73


# Vulnerability details

## Impact

Claiming of the ETH native currency requires `token` to be set to `bribeVault`. If the `bribeVault` is modified in `setBribeVault()` then users who have ETH rewards will now be considered to have `ERC20(bribeVault)` tokens. Since `bribeVault` is not an ERC20 token the `transfer()` call will fail and the users will not be able to claim their funds.


## Recommended Mitigation Steps

Consider removing the functionality to change the `bribeVault` or ensuring all funds have been withdraw i.e. `balanceOf(address(this)) == 0` before changing the `bribeVault`.

