## Tags

- bug
- 2 (Med Risk)
- resolved
- sponsor confirmed

# [Arbitrary contract call within `UniV3LpVault._swap` with controllable `swapPath`](https://github.com/code-423n4/2022-04-dualityfocus-findings/issues/31) 

# Lines of code

https://github.com/code-423n4/2022-04-dualityfocus/blob/main/contracts/vault_and_oracles/UniV3LpVault.sol#L621
https://github.com/code-423n4/2022-04-dualityfocus/blob/main/contracts/vault_and_oracles/UniV3LpVault.sol#L379
https://github.com/code-423n4/2022-04-dualityfocus/blob/main/contracts/vault_and_oracles/UniV3LpVault.sol#L520
https://github.com/code-423n4/2022-04-dualityfocus/blob/main/contracts/vault_and_oracles/UniV3LpVault.sol#L521


# Vulnerability details

## Impact

`UniV3LpVault._swap` utilizes `swapRouter.exactInput` to perform swaps between two tokens. During swaps, `transfer` function of each token along the path will be called to propagate the assets.

Since anyone can create a uniswap pair of arbitrary assets, it is possible to include intermediate hop with malicious tokens within the path. Thus `UniV3LpVault._swap` effectively grants users the ability to perform arbitrary contract calls during the swap process if `swapPath` is not validated properly.

Usage of invalidated `swapPath` can be found in `UniV3LpVault.flashFocusCall` and `UniV3LpVault.repayDebt`.

## Proof of Concept

The security of `Comptroller` and `UniV3LpVault` relies on validating all used tokens thoroughly. This is done by a whitelist mechanism where admin decides a predefined set of usable tokens, and users can only perform actions within the allowed range. This whitelist approach eliminates most of the attack surface regarding directly passing in malicious tokens as arguments.

Apart from passing malicious tokens directly, there are a few other potential weaknesses, the most obvious one is leveraging flash loans for collaterals. However, due to the adoption of AAVE LendingPool, the external validation within flash loan pool blocks this approach.

Unfortunately, a more obscure path exists. Looking at the swapping mechanism, it is not hard to realize it is backed by uniswapV3. An interesting characteristic of uniswap pools is that anyone can create pools for any token pairs, thus if we don't fully validate each and every pool we are using, chances are there will be malicious entries hidden within them.

This is partially the case which we see here, the user gets to supply a path, where the source and target are validated against benign tokens, the intermediate ones are not. An example of utilizing path for arbitrary function call is illustrated below
1. Create malicious token tokenM
2. Create pools tokenS<->tokenM and tokenM<->tokenT where tokenS and tokenT are benign tokens
3. Supply path (tokenS, tokenM, tokenT) for swapping

In the above case, when transferring tokenM while doing swap, we have full control over code executed and can insert arbitrary contract calls within.

Noticeably, while gaining arbitrary contract calls sounds dangerous, it does not necessarily mean the contract is exploitable. It still depends on the scenario in which an arbitrary call happens.

In the case of duality, the two locations where arbitrary `swapPath` can be provided is in `flashFocusCall` and `repayDebt`, both in which holds a local lock over `UniV3LpVault`. No global are applied to `Comptroller` or `Ctokens` while performing swaps.

```
    function flashFocusCall(FlashFocusParams calldata params) external override {
        ...
        {
            ...
            if (!tokenOfPool && params.swapPath.length > 0) amountIn0 = _swap(params.swapPath, params.amount);
            ...
        }
        ...
    }

    function flashFocus(FlashFocusParams calldata params)
        external
        override
        nonReentrant(true)
        isAuthorizedForToken(params.tokenId)
        avoidsShortfall
    {
        ...
        flashLoan.LENDING_POOL().flashLoan(
            receiverAddress,
            assets,
            amounts,
            modes,
            onBehalfOf,
            newParams,
            referralCode
        );
    }

    function repayDebt(RepayDebtParams calldata params)
        external
        override
        nonReentrant(true)
        isAuthorizedForToken(params.tokenId)
        avoidsShortfall
        returns (uint256 amountReturned)
    {
        ...
        {
            ...
            if (amountOutFrom0 == 0 && params.swapPath0.length > 0) amountOutFrom0 = _swap(params.swapPath0, amount0);
            if (amountOutFrom1 == 0 && params.swapPath1.length > 0) amountOutFrom1 = _swap(params.swapPath1, amount1);
            ...
        }
        ...
    }
```


The lack of global locks here had us doubting whether an attack is possible. While we spent a considerable amount of time and failed to come up with any possible attack vectors, the complexity of the system held us back from concluding that an attack is impossible.

Thus we report this finding here in hope of inspiring developers either to prove the attack impossible or mitigate the attack surface.

## Tools Used

vim, ganache-cli

## Recommended Mitigation Steps

The easiest way to mitigate this is to validate the entire path against a predefined whitelist while in `_checkSwapPath`. This approach is far from optimal and also limits the flexibility of swapping between tokens. However, before security is proved, this is the best approach we can come up with.


