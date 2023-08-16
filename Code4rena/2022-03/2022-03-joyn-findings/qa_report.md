## Tags

- bug
- sponsor confirmed
- QA (Quality Assurance)

# [QA Report](https://github.com/code-423n4/2022-03-joyn-findings/issues/131) 

* different pragma versions - the core-contracts use `pragma solidity ^0.8.0` and the rest of the contracts use `pragma solidity ^0.8.4`

* use a specific solidity version instead of using `^`, to prevent future solidity versions impacting your code and creating issues.

* In the comments and variable names you wrote ETH instead of wETH, which is un-correct (that's an ERC20 so it must be wETH)
```sol
function transferSplitAsset(address to, uint256 value)
    private
    returns (bool didSucceed)
{
    // Try to transfer ETH to the given recipient.
    didSucceed = IERC20(splitAsset).transfer(to, value);
    require(didSucceed, "Failed to transfer ETH");
    emit TransferETH(to, value, didSucceed);
}
```

* In the comment before the function, you wrote returns instead of the known `@return` tag
```sol
/**
 * @notice Mint token
 * @dev A starting index is calculated at the time of first mint
 * returns a tokenId
 * @param _to Token recipient
 */
function mint(address _to) private returns (uint256 tokenId) {
    if (startingIndex == 0) {
        setStartingIndex();
    }
    tokenId = ((startingIndex + totalSupply()) % maxSupply) + 1;
    _mint(_to, tokenId);
}
```

* Low level calls (call, delegate call and static call) return success if the called contract doesn’t exist (not deployed or destructed), can be seen here https://github.com/Uniswap/v3-core/blob/main/audits/tob/audit.pdf (report #9) and here https://docs.soliditylang.org/en/develop/control-structures.html#error-handling-assert-require-revert-and-exceptions. 
That means that in `attemptETHTransfer`, if `to` doesn't exist the call will fail but success will be set to true, which will act like the call was successful.
```sol
function attemptETHTransfer(address to, uint256 value)
    private
    returns (bool)
    {
        // Here increase the gas limit a reasonable amount above the default, and try
        // to send ETH to the recipient.
        // NOTE: This might allow the recipient to attempt a limited reentrancy attack.
        (bool success, ) = to.call{value: value, gas: 30000}("");
        return success;
    }
```

* add `onlyUnInitialized` modifier to the `initialize` function, otherwise the owner can initialize the contract more than one time

* `HASHED_PROOF` - upper case variable name that is not constant

* if `startingIndex + totalSupply()` will reach `type(uint256).max` the system will be in a stuck state, that's because the calculation in the _mint function will overflow

* contracts not declaring that they implement their interfaces - for example `CoreCollection` and `CoreFactory` don't declare that they implement `ICoreCollection` and `ICoreFactory`

* `ICoreFactory` is imported but not used in `CoreProxy`

* didn't check that the address of the given vault is not zero in the `setPlatformFee` function

* wrong comment in `RoyaltyVaultFactory` and `SplitFactory`
```sol
/**
 * @dev Set Platform fee for collection contract.
 * @param _platformFee Platform fee in scaled percentage. (5% = 200)
 * @param _vault vault address.
 */
function setPlatformFee(address _vault, uint256 _platformFee) external {
    IRoyaltyVault(_vault).setPlatformFee(_platformFee);
}

/**
 * @dev Set Platform fee recipient for collection contract.
 * @param _vault vault address.
 * @param _platformFeeRecipient Platform fee recipient.
 */
function setPlatformFeeRecipient(
    address _vault,
    address _platformFeeRecipient
) external {
    require(_vault != address(0), "Invalid vault");
    require(
        _platformFeeRecipient != address(0),
        "Invalid platform fee recipient"
    );
    IRoyaltyVault(_vault).setPlatformFeeRecipient(_platformFeeRecipient);
}
```