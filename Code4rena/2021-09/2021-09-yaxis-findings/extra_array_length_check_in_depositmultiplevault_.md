## Tags

- bug
- G (Gas Optimization)
- sponsor acknowledged
- sponsor confirmed

# [extra array length check in depositMultipleVault ](https://github.com/code-423n4/2021-09-yaxis-findings/issues/20) 

# Handle

gpersoon


# Vulnerability details

## Impact
The function depositMultipleVault of VaultHelper doesn't check the array size of _tokens and _amounts are the same length.
In previous version of solidity there were bugs with giving an enormous large array to a function which accepted memory arrays. 
Although depositMultipleVault uses calldata arrays, it is probably better to add a check on the length.

On the other hand the function depositMultiple of Vault.sol does check it.

## Proof of Concept
https://github.com/code-423n4/2021-09-yaxis/blob/main/contracts/v3/VaultHelper.sol#L57
```JS
 function depositMultipleVault(
        address _vault,
        address[] calldata _tokens,
        uint256[] calldata _amounts
    )  external {
        for (uint8 i = 0; i < _amounts.length; i++) {
        ...
```

https://github.com/code-423n4/2021-09-yaxis/blob/main/contracts/v3/Vault.sol#L188
```JS
 function depositMultiple(
        address[] calldata _tokens,
        uint256[] calldata _amounts
    )  external override notHalted returns (uint256 _shares) {
        require(_tokens.length == _amounts.length, "!length");

        for (uint8 i; i < _amounts.length; i++) {
           ... 
```


## Tools Used

## Recommended Mitigation Steps
Add something like the following in depositMultipleVault:
   require(_tokens.length == _amounts.length, "!length");

