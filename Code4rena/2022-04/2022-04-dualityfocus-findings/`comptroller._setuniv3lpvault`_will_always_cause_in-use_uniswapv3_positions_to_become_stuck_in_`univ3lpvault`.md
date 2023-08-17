## Tags

- bug
- 2 (Med Risk)
- resolved
- sponsor confirmed

# [`Comptroller._setUniV3LpVault` will always cause in-use uniswapV3 positions to become stuck in `UniV3LpVault`](https://github.com/code-423n4/2022-04-dualityfocus-findings/issues/28) 

# Lines of code

https://github.com/code-423n4/2022-04-dualityfocus/blob/main/contracts/compound_rari_fork/Comptroller.sol#L1105


# Vulnerability details

## Impact

`Comptroller._setUniV3LpVault` allows the admin of `Comptroller` to change the accompanying `UniV3LpVault`. However since actions including collateral calculation, uniswapV3 position withdrawal, uniswapV3 collateral liquidation all require `Comptroller` and `UniV3LpVault` to cooperate seamlessly, a change in `Comptroller.uniV3LpVault` would mean all the above actions are no longer performable on existing NFTs.

## Proof of Concept

`_setUniV3LpVault` allows changing of `uniV3LpVault`.

```
    function _setUniV3LpVault(IUniV3LpVault newVault) public returns (uint256) {
        ...
        uniV3LpVault = newVault;
        ...
    }
```

However, functions such as `UniV3LpVault.withdrawToken` require `Comptroller` to estimate NFT collateral value. This estimation can only be done when the address of `UniV3LpVault` matches `Comptroller.uniV3LpVault` as shown in `addNFTCollateral` below.

```
contract UniV3LpVault is IUniV3LpVault {
    ...
    function withdrawToken(...) external override nonReentrant(false) avoidsShortfall {
        ...
    }

    modifier avoidsShortfall() {
        _;
        (, , uint256 shortfall) = comptroller.getAccountLiquidity(msg.sender);
        require(shortfall == 0, "insufficient liquidity");
    }
    ...
}

contract Comptroller is ComptrollerV3Storage, ComptrollerInterface, ComptrollerErrorReporter, Exponential {
    ...
    function getAccountLiquidity(address account) public view returns (...) {
        (Error err, uint256 liquidity, uint256 shortfall) = getHypotheticalAccountLiquidityInternal(...);
        ...
    }

    function getHypotheticalAccountLiquidityInternal(...) internal view returns (...) {
        ...
        addNFTCollateral(account, vars);
        ...
    }

    function addNFTCollateral(address account, AccountLiquidityLocalVars memory vars) internal view {
        uint256 userTokensLength = uniV3LpVault.getUserTokensLength(account);
        for (uint256 i = 0; i < userTokensLength; i++) {
            ...
            {
                ...
                address poolAddress = uniV3LpVault.getPoolAddress(tokenId);
                ...
            }
            ...
        }
        ...
    }
}
```
The mutual reliance causes NFT tokens to become stuck. In some cases users can solve this issue by depositing more collateral to cover the shortcoming caused by "disappearing NFTs". In other cases such as liquidation, the functionality becomes downright broken and unuseable..

## Tools Used

vim, ganache-cli

## Recommended Mitigation Steps

Remove the option to change `Comptroller.uniV3LpVault` altogether, as this functionality is not really helpful for the overall protocol.
Another way to handle this is to forcefully evict all NFTs before changing the vault. However, this is extremely complex as it would potentially cause users to become severely under-collateralized, and would require more care in tracking and maintaining states.


