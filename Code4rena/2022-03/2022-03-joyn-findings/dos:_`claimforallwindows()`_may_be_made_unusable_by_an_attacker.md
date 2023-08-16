## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [DoS: `claimForAllWindows()` May Be Made Unusable By An Attacker](https://github.com/code-423n4/2022-03-joyn-findings/issues/6) 

# Lines of code

https://github.com/code-423n4/2022-03-joyn/blob/c9297ccd925ebb2c44dbc6eaa3effd8db5d2368a/splits/contracts/Splitter.sol#L50-L59


# Vulnerability details

## Impact

When the value of `currentWindow` is raised sufficiently high `Splitter.claimForAllWindows()` will not be able to be called due to the block gas limit.

`currentWindow` can only ever be incremented and thus will always increase. This value will naturally increase as royalties are paid into the contract.

Furthermore, an attacker can continually increment `currentWindow` by calling `incrementWindow()`. An attacker can impersonate a `IRoyaltyVault` and send 1 WEI worth of WETH to pass the required checks.

## Proof of Concept

Excerpt from `Splitter.claimForAllWindows()` demonstrating the for loop over `currentWindow` that will grow indefinitely.
```
        for (uint256 i = 0; i < currentWindow; i++) {
            if (!isClaimed(msg.sender, i)) {
                setClaimed(msg.sender, i);

                amount += scaleAmountByPercentage(
                    balanceForWindow[i],
                    percentageAllocation
                );
            }
        }
```

`Splitter.incrementWindow()` may be called by an attacker increasing `currentWindow`.
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

## Recommended Mitigation Steps

Consider modifying the function `claimForAllWindows()` to instead claim for range of windows. Pass the function a `startWindow` and `endWindow` and only iterate through windows in that range. Ensure that `endWindow < currentWindow`.

