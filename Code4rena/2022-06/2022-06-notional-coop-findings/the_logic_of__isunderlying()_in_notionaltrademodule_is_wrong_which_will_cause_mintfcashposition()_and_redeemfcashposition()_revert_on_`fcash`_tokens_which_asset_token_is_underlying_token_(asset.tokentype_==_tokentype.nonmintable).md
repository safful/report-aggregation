## Tags

- bug
- help wanted
- 2 (Med Risk)
- disagree with severity
- sponsor confirmed
- Index

# [The logic of _isUnderlying() in NotionalTradeModule is wrong which will cause mintFCashPosition() and redeemFCashPosition() revert on `fcash` tokens which asset token is underlying token (asset.tokenType == TokenType.NonMintable)](https://github.com/code-423n4/2022-06-notional-coop-findings/issues/87) 

# Lines of code

https://github.com/code-423n4/2022-06-notional-coop/blob/6f8c325f604e2576e2fe257b6b57892ca181509a/index-coop-notional-trade-module/contracts/protocol/modules/v1/NotionalTradeModule.sol#L558-L575


# Vulnerability details

## Impact
For some `fcash` the asset token is underlying token (`asset.tokenType == TokenType.NonMintable`) and `NotionalV2` will not handle minting or burning when it is called with `useUnderlying==True` for those `fcash`s (according to what I asked from sponsor). In summery most of the logics in `NotionalTradeModule ` will not work for those `fcash` tokens because `_isUnderlying()` returns `true` result for those tokens which would make `NotionalTradeModule`'s logic for `mintFCashPosition()` and `redeemFCashPosition()` will eventually call `redeemToUnderlying()` and `mintViaUnderlying()` in `wfCashLogic` and those function in `wfCashLogic` will call `NotionalV2` with `useUnderlying==True` and `NotionalV2` will fail and revert for `fcash` tokens which asset token is underlying token, so the whole transaction will fail and `_mintFCashPosition()` and `_redeemFCashPosition()`  logic in `NotionalTradeModule ` will not work for those `fcash` tokens and manager can't add them to `set` protocol.


## Proof of Concept
when for some `fcash` asset token is underlying token, all calls to `NotionalV2` should be with `useUnderlying==False`. but `_isUnderlying()` in `NotionalTradeModule` contract first check that `isUnderlying = _paymentToken == underlyingToken` so for `fcash` tokens where asset token is underlying token it is going to return `isUnderlying==True`. let's assume that for some specific `fcash` asset token is underlying token (`asset.tokenType == TokenType.NonMintable`) and follow the code execution. 
This is `_isUnderlying()` code in `NotionalTradeModule`:
```
    function _isUnderlying(
        IWrappedfCashComplete _fCashPosition,
        IERC20 _paymentToken
    )
    internal
    view
    returns(bool isUnderlying)
    {
        (IERC20 underlyingToken, IERC20 assetToken) = _getUnderlyingAndAssetTokens(_fCashPosition);
        isUnderlying = _paymentToken == underlyingToken;
        if(!isUnderlying) {
            require(_paymentToken == assetToken, "Token is neither asset nor underlying token");
        }
    }
```
As you can see it calls `_getUnderlyingAndAssetTokens()` and then check `_paymentToken == underlyingToken` to see that if payment token is equal to `underlyingToken`. `_getUnderlyingAndAssetTokens()` uses `getUnderlyingToken()` and `getAssetToken()` in `wfCashBase`. This is `getUnderlyingToken()` code in `wfCashBase`:
```
    /// @notice Returns the token and precision of the token that this token settles
    /// to. For example, fUSDC will return the USDC token address and 1e6. The zero
    /// address will represent ETH.
    function getUnderlyingToken() public view override returns (IERC20 underlyingToken, int256 underlyingPrecision) {
        (Token memory asset, Token memory underlying) = NotionalV2.getCurrency(getCurrencyId());

        if (asset.tokenType == TokenType.NonMintable) {
            // In this case the asset token is the underlying
            return (IERC20(asset.tokenAddress), asset.decimals);
        } else {
            return (IERC20(underlying.tokenAddress), underlying.decimals);
        }
    }
```
As you can see for our specific `fcash` token this function will return asset token as underlying token. so for this specific `fcash` token, the asset token and underlying token will be same in `_isUnderlying()` of `NationalTradeModule` but because code first check `isUnderlying = _paymentToken == underlyingToken` so the function will return `isUnderlying=True` as a result for our specific `fcash` token (which asset token is underlying token)
This is `_mintFCashPosition()` and `_redeemFCashPosition()` code in `NotionalTradeModule `:
```
    /**
     * @dev Redeem a given fCash position from the specified send token (either underlying or asset token)
     * @dev Alo adjust the components / position of the set token accordingly
     */
    function _mintFCashPosition(
        ISetToken _setToken,
        IWrappedfCashComplete _fCashPosition,
        IERC20 _sendToken,
        uint256 _fCashAmount,
        uint256 _maxSendAmount
    )
    internal
    returns(uint256 sentAmount)
    {
        if(_fCashAmount == 0) return 0;

        bool fromUnderlying = _isUnderlying(_fCashPosition, _sendToken);


        _approve(_setToken, _fCashPosition, _sendToken, _maxSendAmount);

        uint256 preTradeSendTokenBalance = _sendToken.balanceOf(address(_setToken));
        uint256 preTradeReceiveTokenBalance = _fCashPosition.balanceOf(address(_setToken));

        _mint(_setToken, _fCashPosition, _maxSendAmount, _fCashAmount, fromUnderlying);


        (sentAmount,) = _updateSetTokenPositions(
            _setToken,
            address(_sendToken),
            preTradeSendTokenBalance,
            address(_fCashPosition),
            preTradeReceiveTokenBalance
        );

        require(sentAmount <= _maxSendAmount, "Overspent");
        emit FCashMinted(_setToken, _fCashPosition, _sendToken, _fCashAmount, sentAmount);
    }

    /**
     * @dev Redeem a given fCash position for the specified receive token (either underlying or asset token)
     * @dev Alo adjust the components / position of the set token accordingly
     */
    function _redeemFCashPosition(
        ISetToken _setToken,
        IWrappedfCashComplete _fCashPosition,
        IERC20 _receiveToken,
        uint256 _fCashAmount,
        uint256 _minReceiveAmount
    )
    internal
    returns(uint256 receivedAmount)
    {
        if(_fCashAmount == 0) return 0;

        bool toUnderlying = _isUnderlying(_fCashPosition, _receiveToken);
        uint256 preTradeReceiveTokenBalance = _receiveToken.balanceOf(address(_setToken));
        uint256 preTradeSendTokenBalance = _fCashPosition.balanceOf(address(_setToken));

        _redeem(_setToken, _fCashPosition, _fCashAmount, toUnderlying);


        (, receivedAmount) = _updateSetTokenPositions(
            _setToken,
            address(_fCashPosition),
            preTradeSendTokenBalance,
            address(_receiveToken),
            preTradeReceiveTokenBalance
        );


        require(receivedAmount >= _minReceiveAmount, "Not enough received amount");
        emit FCashRedeemed(_setToken, _fCashPosition, _receiveToken, _fCashAmount, receivedAmount);

    }
```
As you can see they both uses `_isUnderlying()` to find out that if `_sendToken` is asset token or underlying token. for our specific `fcash` token, the result of `_isUnderlying()` will be `True` and `_mintFCashPosition()` and `_redeemFCashPosition()`  will call `_mint()` and `_redeem()` with `toUnderlying` set as `True`. This is `_mint()` and `_redeem()` code:
```
    /**
     * @dev Invokes the wrappedFCash token's mint function from the setToken
     */
    function _mint(
        ISetToken _setToken,
        IWrappedfCashComplete _fCashPosition,
        uint256 _maxAssetAmount,
        uint256 _fCashAmount,
        bool _fromUnderlying
    )
    internal
    {
        uint32 minImpliedRate = 0;

        bytes4 functionSelector = 
            _fromUnderlying ? _fCashPosition.mintViaUnderlying.selector : _fCashPosition.mintViaAsset.selector;
        bytes memory mintCallData = abi.encodeWithSelector(
            functionSelector,
            _maxAssetAmount,
            uint88(_fCashAmount),
            address(_setToken),
            minImpliedRate,
            _fromUnderlying
        );
        _setToken.invoke(address(_fCashPosition), 0, mintCallData);
    }

    /**
     * @dev Redeems the given amount of fCash token on behalf of the setToken
     */
    function _redeem(
        ISetToken _setToken,
        IWrappedfCashComplete _fCashPosition,
        uint256 _fCashAmount,
        bool _toUnderlying
    )
    internal
    {
        uint32 maxImpliedRate = type(uint32).max;

        bytes4 functionSelector =
            _toUnderlying ? _fCashPosition.redeemToUnderlying.selector : _fCashPosition.redeemToAsset.selector;
        bytes memory redeemCallData = abi.encodeWithSelector(
            functionSelector,
            _fCashAmount,
            address(_setToken),
            maxImpliedRate
        );
        _setToken.invoke(address(_fCashPosition), 0, redeemCallData);
    }
```
As you can see they are using `_toUnderlying` value to decide calling between (`mintViaUnderlying()` or `mintViaAsset()`) and (`redeemToUnderlying()` or `redeemToAsset()`), for our specific `fcash` `_toUnderlying` will be `True` so those functions will call `mintViaUnderlying()` and `redeemToUnderlying()` in `wfCashLogic`.
`mintViaUnderlying()` and `redeemToUnderlying()` in `wfCashLogic` execution flow eventually would call `NotionalV2` functions with `useUnderlying=True` for this specific `fcash` token, but `NotionalV2` will revert for that call because for that `fcash` token asset token is underlying token and `NotionalV2` can't handle calls with `useUnderlying==True` for that `fcash` Token. This will cause all the transaction to fail and manager can't call `redeemFCashPosition()` or `mintFCashPosition()` functions for those `fcash` tokens that asset token is underlying token.
In summery `NotionalTradeModule` logic will not work for all `fcash` tokens becasue the logic of `_isUnderlying()` is wrong for `fcash` tokens that asset token is underlying token.

## Tools Used
VIM

## Recommended Mitigation Steps
Change the logic of `_isUnderlying()` in `NotionalTradeModule` so it returns correct results for all `fcash` tokens. one simple solution can be that it first check `payment token`  value with `asset token` value.

