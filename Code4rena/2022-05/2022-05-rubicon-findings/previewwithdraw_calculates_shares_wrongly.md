## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [previewWithdraw calculates shares wrongly](https://github.com/code-423n4/2022-05-rubicon-findings/issues/140) 

# Lines of code

https://github.com/code-423n4/2022-05-rubicon/blob/main/contracts/rubiconPools/BathToken.sol#L499


# Vulnerability details

The fee is wrongly accounted for in `previewWithdraw`.

## Impact
Function returns wrong result;
Additionally, `withdraw(assets,to,from)` will always revert. (The user can still withdraw his assets via other functions).

## Proof of Concept
The `previewWithdraw` function returns *less* shares than the required assets (notice the substraction):
```
            uint256 amountWithdrawn;
            uint256 _fee = assets.mul(feeBPS).div(10000);
            amountWithdrawn = assets.sub(_fee);
            shares = convertToShares(amountWithdrawn);
```
This won't work, because if the user wants to receive amount of `assets`, he needs to burn *more* shares than that to account for the fee. Not less.
This will also make `withdraw(assets,to,from)` [revert](https://github.com/code-423n4/2022-05-rubicon/blob/main/contracts/rubiconPools/BathToken.sol#L514:#L519), because it takes the amount of shares from `previewWithdraw`, and then checks how much assets were really sent to the user, and verifies that it's at least how much he asked for:
```
        uint256 expectedShares = previewWithdraw(assets);
        uint256 assetsReceived = _withdraw(expectedShares, receiver);
        require(assetsReceived >= assets, "You cannot withdraw the amount of assets you expected");
```
But since the expectedShares is smaller than the original amount, and since `_withdraw` [deducts](https://github.com/code-423n4/2022-05-rubicon/blob/main/contracts/rubiconPools/BathToken.sol#L604) the fee from expectedShares, then always `assets > assetsReceived`, and the function will revert.

## Recommended Mitigation Steps
The amount of shares that `previewWithdraw` should return is:
`convertToShares(assets.add(assets.mul(feeBPS).div((10000.sub(feeBPS))))`
I prove this mathematically in [this](https://i.ibb.co/hX41vzV/c4wd.jpg) image.

