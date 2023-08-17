## Tags

- bug
- 2 (Med Risk)
- primary issue
- satisfactory
- selected for report
- sponsor confirmed
- M-20

# [smartWallet address is not guaranteed correct. ETH may be lost](https://github.com/code-423n4/2022-11-stakehouse-findings/issues/317) 

# Lines of code

https://github.com/code-423n4/2022-11-stakehouse/blob/main/contracts/liquid-staking/SavETHVault.sol#L206-L207
https://github.com/code-423n4/2022-11-stakehouse/blob/main/contracts/liquid-staking/SavETHVault.sol#L209


# Vulnerability details

## Impact
Liquid staking manager call `function withdrawETHForStaking(address _smartWallet, uint256 _amount)` to withdraw ETH for staking. It's manager's responsibility to set the correct `_smartWallet` address. However, there is no way to guarantee this. If a typo (or any other reasons) leads to a non-zero non-existent `_smartWallet` address, this function won't be able to detect the problem, and the [ETH transfer statement](https://github.com/code-423n4/2022-11-stakehouse/blob/main/contracts/liquid-staking/SavETHVault.sol#L209) will always return `true`. This will result in the ETH permanently locked to a non-existent account.

## Proof of Concept
Liquid staking manager call `function withdrawETHForStaking(address _smartWallet, uint256 _amount)` with a non-zero non-existent `_smartWallet` address and some `_amount` of ETH. Function call will succeed but the ETH will be locked to the non-existent `_smartWallet` address.

## Tools Used
Manual audit.

## Recommended Mitigation Steps
The problem can be solved if we can verify the `_smartWallet` is a valid existent smartWallet before ETH transfer. The easiest solution is to verify the smartWallet has a valid owner since the smart wallet we are using is ownable. So, just add the checking owner code before [ETH transfer](https://github.com/code-423n4/2022-11-stakehouse/blob/main/contracts/liquid-staking/SavETHVault.sol#L209).