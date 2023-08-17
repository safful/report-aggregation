## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed
- Index

# [Residual Allowance Might Allow Tokens In SetToken To Be Stolen](https://github.com/code-423n4/2022-06-notional-coop-findings/issues/160) 

# Lines of code

https://github.com/code-423n4/2022-06-notional-coop/blob/6f8c325f604e2576e2fe257b6b57892ca181509a/index-coop-notional-trade-module/contracts/protocol/modules/v1/NotionalTradeModule.sol#L418
https://github.com/code-423n4/2022-06-notional-coop/blob/6f8c325f604e2576e2fe257b6b57892ca181509a/index-coop-notional-trade-module/contracts/protocol/modules/v1/NotionalTradeModule.sol#L493


# Vulnerability details

## Proof-of-Concept

Whenever `_mintFCashPosition` function is called to mint new fCash position, the contract will call the `_approve` function to set the allowance to `_maxSendAmount` so that the fCash Wrapper contact can pull the payment tokens from the SetToken contract during minting.

[https://github.com/code-423n4/2022-06-notional-coop/blob/6f8c325f604e2576e2fe257b6b57892ca181509a/index-coop-notional-trade-module/contracts/protocol/modules/v1/NotionalTradeModule.sol#L418](https://github.com/code-423n4/2022-06-notional-coop/blob/6f8c325f604e2576e2fe257b6b57892ca181509a/index-coop-notional-trade-module/contracts/protocol/modules/v1/NotionalTradeModule.sol#L418)

```solidity
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

    _mint(_setToken, _fCashPosition, _maxSendAmount, _fCashAmount, fromUnderlying)

	..SNIP..
}
```

Note that `_maxSendAmount` is the maximum amount of payment tokens that is allowed to be consumed during minting. This is not the actual amount of payment tokens consumed during the minting process. Thus, after the minting, there will definitely be some residual allowance since it is unlikely that the fCash wrapper contract will consume the exact maximum amount during minting.

The following piece of code shows that having some residual allowance is expected. The `_approve` function will not set the allowance unless there is insufficient allowance.

[https://github.com/code-423n4/2022-06-notional-coop/blob/6f8c325f604e2576e2fe257b6b57892ca181509a/index-coop-notional-trade-module/contracts/protocol/modules/v1/NotionalTradeModule.sol#L493](https://github.com/code-423n4/2022-06-notional-coop/blob/6f8c325f604e2576e2fe257b6b57892ca181509a/index-coop-notional-trade-module/contracts/protocol/modules/v1/NotionalTradeModule.sol#L493)

```solidity
/**
 * @dev Approve the given wrappedFCash instance to spend the setToken's sendToken 
 */
function _approve(
    ISetToken _setToken,
    IWrappedfCashComplete _fCashPosition,
    IERC20 _sendToken,
    uint256 _maxAssetAmount
)
internal
{
    if(IERC20(_sendToken).allowance(address(_setToken), address(_fCashPosition)) < _maxAssetAmount) {
        bytes memory approveCallData = abi.encodeWithSelector(_sendToken.approve.selector, address(_fCashPosition), _maxAssetAmount);
        _setToken.invoke(address(_sendToken), 0, approveCallData);
    }
}
```

## Impact

Having residual allowance increases the risk of the asset tokens being stolen from the SetToken contract. SetToken contract is where all the tokens/assets are held. If the Notional's fCash wrapper contract is compromised, it will allow the compromised fCash wrapper contract to withdraw funds from the SetToken contract due to the residual allowance. 

Note that Notional's fCash wrapper contract is not totally immutable, as it is a upgradeable contract. This is an additional risk factor to be considered. If the Notional's deployer account is compromised, the attacker could upgrade the Notional's fCash wrapper contract to a malicious one to withdraw funds from the Index Coop's SetToken contract due to the residual allowance. 

Index Coop and Notional are two separate protocols and teams. Thus, it is a good security practice not to place any trust on external party wherever possible to ensure that if one party is compromised, it won't affect the another party. Thus, there should not be any residual allowance that allows Notional's contract to withdraw fund from Index Coop's contract in any circumstance.

In the worst case scenario, a "lazy" manager might simply set the `_maxAssetAmount` to `type(uint256).max`. Thus, this will result in large amount of residual allowance left, and expose the SetToken contract to significant risk.

## Recommended Mitigation Steps

Approve the allowance on-demand whenever _`mintFCashPosition` is called, and reset the allowance back to zero after each minting process to eliminate any residual allowance.

```diff
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

    _mint(_setToken, _fCashPosition, _maxSendAmount, _fCashAmount, fromUnderlying)

	..SNIP..
	
+	// Reset the allowance back to zero after minting
+	_approve(_setToken, _fCashPosition, _sendToken, 0);
}
```

Update the `_approve` accordingly to remove the if-statement related to residual allowance.

```diff
function _approve(
    ISetToken _setToken,
    IWrappedfCashComplete _fCashPosition,
    IERC20 _sendToken,
    uint256 _maxAssetAmount
)
internal
{
-    if(IERC20(_sendToken).allowance(address(_setToken), address(_fCashPosition)) < _maxAssetAmount) {
        bytes memory approveCallData = abi.encodeWithSelector(_sendToken.approve.selector, address(_fCashPosition), _maxAssetAmount);
        _setToken.invoke(address(_sendToken), 0, approveCallData);
-    }
}
```

