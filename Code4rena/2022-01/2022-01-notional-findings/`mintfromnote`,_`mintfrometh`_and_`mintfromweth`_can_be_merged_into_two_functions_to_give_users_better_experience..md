## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [`mintFromNOTE`, `mintFromETH` and `mintFromWETH` can be merged into two functions to give users better experience.](https://github.com/code-423n4/2022-01-notional-findings/issues/41) 

# Handle

TomFrenchBlockchain


# Vulnerability details

## Impact
Greater flexibility for users and better conversion between a mix of ETH/NOTE and sNOTE.

## Proof of Concept

`sNOTE` allows users to provider either NOTE, ETH or WETH to provide liquidity in return for BPT to mint sNOTE with through the functions `mintFromNOTE`, `mintFromETH` and `mintFromWETH`.

https://github.com/code-423n4/2022-01-notional/blob/d171cad9e86e0d02e0909eb66d4c24ab6ea6b982/contracts/sNOTE.sol#L146-L188

Balancer allows users to deposit multiple assets at once so the same functionality while being more flexible (by allowing deposits in both NOTE and ETH at the same time) and giving users better execution (A user depositing NOTE and ETH together gets more SNOTE than one who deposits NOTE and then ETH afterwards)

Consider the code snippet:

```
/// @notice Mints sNOTE from some amount of NOTE and ETH
/// @param noteAmount amount of NOTE to transfer into the sNOTE contract
function mintFromETH(uint256 noteAmount) payable external nonReentrant {
    IAsset[] memory assets = new IAsset[](2);
    assets[0] = IAsset(address(0));
    assets[1] = IAsset(address(NOTE));
    uint256[] memory maxAmountsIn = new uint256[](2);
    maxAmountsIn[0] = msg.value;
    maxAmountsIn[1] = noteAmount;

    _mintFromAssets(assets, maxAmountsIn);
}

/// @notice Mints sNOTE from some amount of NOTE and WETH
/// @param wethAmount amount of WETH to transfer into the sNOTE contract
/// @param noteAmount amount of NOTE to transfer into the sNOTE contract
function mintFromWETH(uint256 wethAmount, uint256 noteAmount) external nonReentrant {
    // Transfer the WETH and NOTE balance into sNOTE first
    WETH.safeTransferFrom(msg.sender, address(this), wethAmount);
    NOTE.safeTransferFrom(msg.sender, address(this), noteAmount);

    IAsset[] memory assets = new IAsset[](2);
    assets[0] = IAsset(address(WETH));
    assets[1] = IAsset(address(NOTE));
    uint256[] memory maxAmountsIn = new uint256[](2);
    maxAmountsIn[0] = wethAmount;
    maxAmountsIn[1] = noteAmount;

    _mintFromAssets(assets, maxAmountsIn);
}
```

## Recommended Mitigation Steps

Replace current functions with above versions

