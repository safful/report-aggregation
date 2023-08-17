## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed
- Notional

# [deposit() and mint() and _redeemInternal() in wfCashERC4626() will revert for all fcash that asset token is underlying token because they always call _mintInternal() with useUnderlying==True](https://github.com/code-423n4/2022-06-notional-coop-findings/issues/82) 

# Lines of code

https://github.com/code-423n4/2022-06-notional-coop/blob/6f8c325f604e2576e2fe257b6b57892ca181509a/notional-wrapped-fcash/contracts/wfCashERC4626.sol#L177-L184
https://github.com/code-423n4/2022-06-notional-coop/blob/6f8c325f604e2576e2fe257b6b57892ca181509a/notional-wrapped-fcash/contracts/wfCashERC4626.sol#L168-L175
https://github.com/code-423n4/2022-06-notional-coop/blob/6f8c325f604e2576e2fe257b6b57892ca181509a/notional-wrapped-fcash/contracts/wfCashERC4626.sol#L225-L241


# Vulnerability details

## Impact
For some `fcash` the asset token is underlying token (`asset.tokenType == TokenType.NonMintable`) and `NotionalV2` will not handle minting with `useUnderlying==True` for those `fcash`s (according to what I asked from sponsor). In summery most of the logics in `wfCashERC4626` will not work for those `fcash` tokens.

when for some `fcash` asset token is underlying token, all calls to `NotionalV2` should be with `useUnderlying==False`. but `deposit()` and `mint()` in `wfCashERC4626` contract call `_mintInternal()` with `useUnderlying==True` and it calls `NotionalV2.batchLend()` with `depositUnderlying==true` so the `NotionV2` call will fail for `fcash` tokens that asset token is underlying token and it would cause  that `deposit()` and `mint()`  logic `wfCashERC4626`  will not work and contract will be useless for those tokens.
`_redeemInternal()` issue is similar and it calls `_burn()` with `redeemToUnderlying: true` which execution eventually calls `NotionalV2.batchBalanceAndTradeAction()` with `toUnderlying=True` which will revert so `_redeemInternal()` will fail and because `withdraw()` and `redeem` use it, so they will not work too for those `fcash` tokens that asset token is underlying token.

## Proof of Concept
This is `deposit()` and `mint()`  code in `wfCashERC4626`:
```
    /** @dev See {IERC4626-deposit} */
    function deposit(uint256 assets, address receiver) public override returns (uint256) {
        uint256 shares = previewDeposit(assets);
        // Will revert if matured
        _mintInternal(assets, _safeUint88(shares), receiver, 0, true);
        emit Deposit(msg.sender, receiver, assets, shares);
        return shares;
    }

    /** @dev See {IERC4626-mint} */
    function mint(uint256 shares, address receiver) public override returns (uint256) {
        uint256 assets = previewMint(shares);
        // Will revert if matured
        _mintInternal(assets, _safeUint88(shares), receiver, 0, true);
        emit Deposit(msg.sender, receiver, assets, shares);
        return assets;
    }
```
As you can see they both call `_mintInternal()` with last parameter as `true` which is `useUnderlying`'s value. This is `_mintInternal()` code:
```
    function _mintInternal(
        uint256 depositAmountExternal,
        uint88 fCashAmount,
        address receiver,
        uint32 minImpliedRate,
        bool useUnderlying
    ) internal nonReentrant {
        require(!hasMatured(), "fCash matured");
        (IERC20 token, bool isETH) = getToken(useUnderlying);
        uint256 balanceBefore = isETH ? address(this).balance : token.balanceOf(address(this));

        // If dealing in ETH, we use WETH in the wrapper instead of ETH. NotionalV2 uses
        // ETH natively but due to pull payment requirements for batchLend, it does not support
        // ETH. batchLend only supports ERC20 tokens like cETH or aETH. Since the wrapper is a compatibility
        // layer, it will support WETH so integrators can deal solely in ERC20 tokens. Instead of using
        // "batchLend" we will use "batchBalanceActionWithTrades". The difference is that "batchLend"
        // is more gas efficient (does not require and additional redeem call to asset tokens). If using cETH
        // then everything will proceed via batchLend.
        if (isETH) {
            IERC20((address(WETH))).safeTransferFrom(msg.sender, address(this), depositAmountExternal);
            WETH.withdraw(depositAmountExternal);

            BalanceActionWithTrades[] memory action = EncodeDecode.encodeLendETHTrade(
                getCurrencyId(),
                getMarketIndex(),
                depositAmountExternal,
                fCashAmount,
                minImpliedRate
            );
            // Notional will return any residual ETH as the native token. When we _sendTokensToReceiver those
            // native ETH tokens will be wrapped back to WETH.
            NotionalV2.batchBalanceAndTradeAction{value: depositAmountExternal}(address(this), action);
        } else {
            // Transfers tokens in for lending, Notional will transfer from this contract.
            token.safeTransferFrom(msg.sender, address(this), depositAmountExternal);

            // Executes a lending action on Notional
            BatchLend[] memory action = EncodeDecode.encodeLendTrade(
                getCurrencyId(),
                getMarketIndex(),
                fCashAmount,
                minImpliedRate,
                useUnderlying
            );
            NotionalV2.batchLend(address(this), action);
        }

        // Mints ERC20 tokens for the receiver, the false flag denotes that we will not do an
        // operatorAck
        _mint(receiver, fCashAmount, "", "", false);

        _sendTokensToReceiver(token, msg.sender, isETH, balanceBefore);
    }
```
As you can see it calls `NotionalV2` functions with `useUnderlying=True` but according to sponsor clarification `NotionalV2` would fail and revert for those calls because `useUnderlying=True` and `fcash`'s asset token is underlying token (`asset.tokenType == TokenType.NonMintable`).
So in summery for `fcash` tokens which asset token is underlying token `NotionalV2` won't handle calls which include `useUnderlying==True` but in `wfCashERC4626` contract functions like `deposit()`, `mint()`, `withdraw()` and `redeem()` they all uses `useUnderlying==True` always so `wfCashERC4626` won't work for those specific type of tokens which asset token is underlying token(`asset.tokenType == TokenType.NonMintable`)

the detail explanations for functions `withdraw()` and `redeem()` are similar.

## Tools Used
VIM

## Recommended Mitigation Steps
check that if for that `fcash` token asset token  is underlying token or not and set `useUnderlying` based on that.

