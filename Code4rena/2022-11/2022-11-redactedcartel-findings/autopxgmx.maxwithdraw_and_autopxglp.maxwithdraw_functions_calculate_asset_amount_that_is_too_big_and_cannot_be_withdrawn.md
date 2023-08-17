## Tags

- bug
- 2 (Med Risk)
- satisfactory
- selected for report
- sponsor confirmed
- edited-by-warden
- M-04

# [AutoPxGmx.maxWithdraw and AutoPxGlp.maxWithdraw functions calculate asset amount that is too big and cannot be withdrawn](https://github.com/code-423n4/2022-11-redactedcartel-findings/issues/97) 

# Lines of code

https://github.com/code-423n4/2022-11-redactedcartel/blob/03b71a8d395c02324cb9fdaf92401357da5b19d1/src/vaults/PirexERC4626.sol#L225
https://github.com/code-423n4/2022-11-redactedcartel/blob/03b71a8d395c02324cb9fdaf92401357da5b19d1/src/vaults/AutoPxGmx.sol#L14
https://github.com/code-423n4/2022-11-redactedcartel/blob/03b71a8d395c02324cb9fdaf92401357da5b19d1/src/vaults/AutoPxGlp.sol#L14
https://github.com/code-423n4/2022-11-redactedcartel/blob/03b71a8d395c02324cb9fdaf92401357da5b19d1/src/vaults/AutoPxGmx.sol#L315
https://github.com/code-423n4/2022-11-redactedcartel/blob/03b71a8d395c02324cb9fdaf92401357da5b19d1/src/vaults/AutoPxGmx.sol#L199
https://github.com/code-423n4/2022-11-redactedcartel/blob/03b71a8d395c02324cb9fdaf92401357da5b19d1/src/vaults/AutoPxGmx.sol#L215-L216
https://github.com/code-423n4/2022-11-redactedcartel/blob/03b71a8d395c02324cb9fdaf92401357da5b19d1/src/vaults/AutoPxGmx.sol#L332


# Vulnerability details

## Impact
The ERC-4626 Tokenized Vault Standard requires the `maxWithdraw` function to be implemented ([https://ethereum.org/en/developers/docs/standards/tokens/erc-4626/#maxwithdraw](https://ethereum.org/en/developers/docs/standards/tokens/erc-4626/#maxwithdraw)).  

This function is supposed to return "the maximum amount of underlying assets that can be withdrawn from the owner balance with a single withdraw call".  

The `PirexERC4626` contract implements this function ([https://github.com/code-423n4/2022-11-redactedcartel/blob/03b71a8d395c02324cb9fdaf92401357da5b19d1/src/vaults/PirexERC4626.sol#L225](https://github.com/code-423n4/2022-11-redactedcartel/blob/03b71a8d395c02324cb9fdaf92401357da5b19d1/src/vaults/PirexERC4626.sol#L225)).  
It is implemented correctly when `PirexERC4626` is used on its own.  

However in this project, the `PirexERC4626` contract is not used on its own but inherited by `AutoPxGmx` ([https://github.com/code-423n4/2022-11-redactedcartel/blob/03b71a8d395c02324cb9fdaf92401357da5b19d1/src/vaults/AutoPxGmx.sol#L14](https://github.com/code-423n4/2022-11-redactedcartel/blob/03b71a8d395c02324cb9fdaf92401357da5b19d1/src/vaults/AutoPxGmx.sol#L14)) and `AutoPxGlp` ([https://github.com/code-423n4/2022-11-redactedcartel/blob/03b71a8d395c02324cb9fdaf92401357da5b19d1/src/vaults/AutoPxGlp.sol#L14](https://github.com/code-423n4/2022-11-redactedcartel/blob/03b71a8d395c02324cb9fdaf92401357da5b19d1/src/vaults/AutoPxGlp.sol#L14)).  

`AutoPxGmx` and `AutoPxGlp` implement a `withdrawalPenalty` i.e. a fee that is paid when a user wants to withdraw assets from the vault.  

`AutoPxGmx` and `AutoPxGlp` do not override the `maxWithdraw` function.  

This causes the `maxWithdraw` function to return an amount of assets that is too big to be withdrawn.  

So when `maxWithdraw` is called and with the returned amount `withdraw` is called, the call to `withdraw` will revert.  

This can cause issues in any upstream components that rely on `AutoPxGmx` and `AutoPxGlp` to correctly implement the ERC4626 standard.  

For example an upstream wrapper might only allow withdrawals with the maximum amount and determine this maximum amount by calling the `maxWithdraw` function. As this function returns a value that is too big, no withdrawals will be possible.  

## Proof of Concept
1. The `maxWithdraw` function in a `AutoPxGmx` contract is called
2. Now the `withdraw` function is called with the value that was returned by the `maxWithdraw` function ([https://github.com/code-423n4/2022-11-redactedcartel/blob/03b71a8d395c02324cb9fdaf92401357da5b19d1/src/vaults/AutoPxGmx.sol#L315](https://github.com/code-423n4/2022-11-redactedcartel/blob/03b71a8d395c02324cb9fdaf92401357da5b19d1/src/vaults/AutoPxGmx.sol#L315))
3. The `withdraw` function in turn calls the `previewWithdraw` function ([https://github.com/code-423n4/2022-11-redactedcartel/blob/03b71a8d395c02324cb9fdaf92401357da5b19d1/src/vaults/AutoPxGmx.sol#L199](https://github.com/code-423n4/2022-11-redactedcartel/blob/03b71a8d395c02324cb9fdaf92401357da5b19d1/src/vaults/AutoPxGmx.sol#L199))
4. The `previewWithdraw` function will increase the amount of shares to include the `withdrawalPenalty` ([https://github.com/code-423n4/2022-11-redactedcartel/blob/03b71a8d395c02324cb9fdaf92401357da5b19d1/src/vaults/AutoPxGmx.sol#L215-L216](https://github.com/code-423n4/2022-11-redactedcartel/blob/03b71a8d395c02324cb9fdaf92401357da5b19d1/src/vaults/AutoPxGmx.sol#L215-L216)) which causes the amount of shares to burn to be too large and the call to `burn` will revert ([https://github.com/code-423n4/2022-11-redactedcartel/blob/03b71a8d395c02324cb9fdaf92401357da5b19d1/src/vaults/AutoPxGmx.sol#L332](https://github.com/code-423n4/2022-11-redactedcartel/blob/03b71a8d395c02324cb9fdaf92401357da5b19d1/src/vaults/AutoPxGmx.sol#L332))

## Tools Used
VSCode

## Recommended Mitigation Steps
In the `AutoPxGmx` and `AutoPxGlp` function implement the `maxWithdraw` function that overrides the function in `PirexERC4626` and takes into account the `withdrawalPenalty`.  

Potential fix:
```solidity
function maxWithdraw(address owner) public view override returns (uint256) {
    uint256 shares = balanceOf(owner);

    // Calculate assets based on a user's % ownership of vault shares
    uint256 assets = convertToAssets(shares);

    uint256 _totalSupply = totalSupply;

    // Calculate a penalty - zero if user is the last to withdraw
    uint256 penalty = (_totalSupply == 0 || _totalSupply - shares == 0)
        ? 0
        : assets.mulDivDown(withdrawalPenalty, FEE_DENOMINATOR);

    // Redeemable amount is the post-penalty amount
    return assets - penalty;
}
```