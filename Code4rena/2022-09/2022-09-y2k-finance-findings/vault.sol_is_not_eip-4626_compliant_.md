## Tags

- bug
- 3 (High Risk)
- high quality report
- resolved
- sponsor confirmed
- selected for report

# [Vault.sol is not EIP-4626 compliant ](https://github.com/code-423n4/2022-09-y2k-finance-findings/issues/47) 

# Lines of code

https://github.com/code-423n4/2022-09-y2k-finance/blob/ac3e86f07bc2f1f51148d2265cc897e8b494adf7/src/Vault.sol#L244-L252
https://github.com/code-423n4/2022-09-y2k-finance/blob/ac3e86f07bc2f1f51148d2265cc897e8b494adf7/src/SemiFungibleVault.sol#L205-L213
https://github.com/code-423n4/2022-09-y2k-finance/blob/ac3e86f07bc2f1f51148d2265cc897e8b494adf7/src/SemiFungibleVault.sol#L237-L239
https://github.com/code-423n4/2022-09-y2k-finance/blob/ac3e86f07bc2f1f51148d2265cc897e8b494adf7/src/SemiFungibleVault.sol#L244-L246
https://github.com/code-423n4/2022-09-y2k-finance/blob/ac3e86f07bc2f1f51148d2265cc897e8b494adf7/src/SemiFungibleVault.sol#L251-L258
https://github.com/code-423n4/2022-09-y2k-finance/blob/ac3e86f07bc2f1f51148d2265cc897e8b494adf7/src/SemiFungibleVault.sol#L263-L270


# Vulnerability details

## Impact

Other protocols that integrate with Y2K may wrongly assume that the functions are EIP-4626 compliant. Thus, it might cause integration problems in the future that can lead to wide range of issues for both parties. 

## Proof of Concept

All official EIP-4626 requirements can be found on it's [official page](https://eips.ethereum.org/EIPS/eip-4626#methods). Non-compliant functions are listed below along with the reason they are not compliant:

The following functions are missing but should be present:
1. mint(uint256, address) returns (uint256)
2. redeem(uint256, address, address) returns (uint256)

The following functions are non-compliant because they don't account for withdraw and deposit locking:
1. maxDeposit
2. maxMint
3. maxWithdraw
4. maxRedeem

All of the above functions should return 0 when their respective functions are disabled (i.e. maxDeposit should return 0 when deposits are disabled)

previewDeposit is not compliant because it must account for fees which it does not

totalAssets is not compliant because it does not always return the underlying managed by the vault because it fails to include the assets paid out during a depeg or the end of the epoch.

## Tools Used

## Recommended Mitigation Steps

All functions listed above should be modified to meet the specifications of EIP-4626