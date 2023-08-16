## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [Splitter: Anyone can call incrementWindow to steal the tokens in the contract](https://github.com/code-423n4/2022-03-joyn-findings/issues/3) 

# Lines of code

https://github.com/code-423n4/2022-03-joyn/blob/main/splits/contracts/Splitter.sol#L149-L169


# Vulnerability details

## Impact
In general, the Splitter contract's incrementWindow function is only called when tokens are transfer to the contract, ensuring that the number of tokens stored in balanceForWindow is equal to the contract balance.
However, anyone can use a fake RoyaltyVault contract to call the incrementWindow function of the Splitter contract, so that the amount of tokens stored in balanceForWindow is greater than the contract balance, after which the verified user can call the claim or claimForAllWindows functions to steal the tokens in the contract.
```
    function incrementWindow(uint256 royaltyAmount) public returns (bool) {
        uint256 wethBalance;

        require(
            IRoyaltyVault(msg.sender).supportsInterface(IID_IROYALTY),
            "Royalty Vault not supported"
        );
        require(
            IRoyaltyVault(msg.sender).getSplitter() == address(this),
            "Unauthorised to increment window"
        );

        wethBalance = IERC20(splitAsset).balanceOf(address(this));
        require(wethBalance >= royaltyAmount, "Insufficient funds");

        require(royaltyAmount > 0, "No additional funds for window");
        balanceForWindow.push(royaltyAmount);
        currentWindow += 1;
        emit WindowIncremented(currentWindow, royaltyAmount);
        return true;
    }
```
## Proof of Concept
https://github.com/code-423n4/2022-03-joyn/blob/main/splits/contracts/Splitter.sol#L149-L169
## Tools Used
None
## Recommended Mitigation Steps
Add the onlyRoyaltyVault modifier to the incrementWindow function of the Splitter contract to ensure that only RoyaltyVault contracts with a specific address can call this function.

