## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [`CoreCollection.setRoyaltyVault` doesn't check `royaltyVault.royaltyAsset` against `payableToken`, resulting in potential permanent lock of `payableTokens` in royaltyVault](https://github.com/code-423n4/2022-03-joyn-findings/issues/73) 

# Lines of code

https://github.com/code-423n4/2022-03-joyn/blob/main/core-contracts/contracts/CoreCollection.sol#L185
https://github.com/code-423n4/2022-03-joyn/blob/main/core-contracts/contracts/ERC721Payable.sol#L50
https://github.com/code-423n4/2022-03-joyn/blob/main/royalty-vault/contracts/RoyaltyVault.sol#L31


# Vulnerability details

## Impact

Each CoreProxy is allowed to be associated with a RoyaltyVault, the latter which would be responsible for collecting minting fees and distributing to beneficiaries. Potential mismatch between token used in CoreProxy and RoyaltyVault might result in minting tokens being permanently stuck in RoyaltyVault.

## Proof of Concept

Each RoyaltyVault can only handle the `royaltyVault.royaltyAsset` token assigned upon creation, if any other kind of tokens are sent to the vault, it would get stuck inside the vault forever.

```
    function sendToSplitter() external override {
        ...
        require(
            IERC20(royaltyAsset).transfer(splitterProxy, splitterShare) == true,
            "Failed to transfer royalty Asset to splitter"
        );
        ...
        require(
            IERC20(royaltyAsset).transfer(
                platformFeeRecipient,
                platformShare
            ) == true,
            "Failed to transfer royalty Asset to platform fee recipient"
        );
        ...
    }
```

Considering that pairing of CoreProxy and RoyaltyVault is not necessarily handled automatically, and can sometimes be manually assigned, and further combined with the fact that once assigned, CoreProxy does not allow modifications of the pairing RoyaltyVault. We can easily conclude that if a CoreProxy is paired with an incompatible RoyaltyVault, the `payableToken` minting fees automatically transferred to RoyaltyVault by `_handlePayment` will get permanently stuck.

```
     function setRoyaltyVault(address _royaltyVault)
         external
         onlyVaultUninitialized
     {
         ...
         royaltyVault = _royaltyVault;
         ...
     }

     function _handlePayment(uint256 _amount) internal {
         address recipient = royaltyVaultInitialized()
             ? royaltyVault
             : address(this);
         payableToken.transferFrom(msg.sender, recipient, _amount);
         ...
     }
```

## Tools Used

vim, ganache-cli

## Recommended Mitigation Steps

While assigning vaults to CoreProxy, check if `payableToken` is the same as `royaltyVault.royaltyAsset`


```
     function setRoyaltyVault(address _royaltyVault)
         external
         onlyVaultUninitialized
     {
         require(
             payableToken == _royaltyVault.royaltyAsset(),
             "CoreCollection : payableToken must be same as royaltyAsset."
         );
         ...
         royaltyVault = _royaltyVault;
         ...
     }
```


