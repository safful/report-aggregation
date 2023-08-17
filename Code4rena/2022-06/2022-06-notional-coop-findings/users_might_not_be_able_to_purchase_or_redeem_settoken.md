## Tags

- bug
- help wanted
- 2 (Med Risk)
- disagree with severity
- sponsor confirmed
- Index

# [Users Might Not Be Able To Purchase Or Redeem SetToken](https://github.com/code-423n4/2022-06-notional-coop-findings/issues/154) 

# Lines of code

https://github.com/code-423n4/2022-06-notional-coop/blob/6f8c325f604e2576e2fe257b6b57892ca181509a/index-coop-notional-trade-module/contracts/protocol/modules/v1/NotionalTradeModule.sol#L309
https://github.com/code-423n4/2022-06-notional-coop/blob/6f8c325f604e2576e2fe257b6b57892ca181509a/index-coop-notional-trade-module/contracts/protocol/modules/v1/NotionalTradeModule.sol#L385


# Vulnerability details

## Proof-of-Concept

Whenever a setToken is issued or redeemed, the `moduleIssueHook` and `moduleRedeemHook` will be triggered. These two hooks will in turn call the `_redeemMaturedPositions` function to ensure that no matured fCash positions remain in the Set by redeeming any matured fCash position.

[https://github.com/code-423n4/2022-06-notional-coop/blob/6f8c325f604e2576e2fe257b6b57892ca181509a/index-coop-notional-trade-module/contracts/protocol/modules/v1/NotionalTradeModule.sol#L309](https://github.com/code-423n4/2022-06-notional-coop/blob/6f8c325f604e2576e2fe257b6b57892ca181509a/index-coop-notional-trade-module/contracts/protocol/modules/v1/NotionalTradeModule.sol#L309)

```solidity
/**
 * @dev Hook called once before setToken issuance
 * @dev Ensures that no matured fCash positions are in the set when it is issued
 */
function moduleIssueHook(ISetToken _setToken, uint256 /* _setTokenAmount */) external override onlyModule(_setToken) {
    _redeemMaturedPositions(_setToken);
}

/**
 * @dev Hook called once before setToken redemption
 * @dev Ensures that no matured fCash positions are in the set when it is redeemed
 */
function moduleRedeemHook(ISetToken _setToken, uint256 /* _setTokenAmount */) external override onlyModule(_setToken) {
    _redeemMaturedPositions(_setToken);
}
```

The `_redeemMaturedPositions` will loop through all its fCash positions and attempts to redeem any fCash position that has already matured. However, if one of the fCash redemptions fails, it will cause the entire function to revert. If this happens, no one could purchase or redeem the setToken because `moduleIssueHook` and `modileRedeemHook` hooks will revert every single time. Thus, the setToken issuance and redemption will stop working entirely and  this setToken can be considered "bricked".

[https://github.com/code-423n4/2022-06-notional-coop/blob/6f8c325f604e2576e2fe257b6b57892ca181509a/index-coop-notional-trade-module/contracts/protocol/modules/v1/NotionalTradeModule.sol#L385](https://github.com/code-423n4/2022-06-notional-coop/blob/6f8c325f604e2576e2fe257b6b57892ca181509a/index-coop-notional-trade-module/contracts/protocol/modules/v1/NotionalTradeModule.sol#L385)

```solidity
/**
 * @dev Redeem all matured fCash positions for the given SetToken
 */
function _redeemMaturedPositions(ISetToken _setToken)
internal
{
    ISetToken.Position[] memory positions = _setToken.getPositions();
    uint positionsLength = positions.length;

    bool toUnderlying = redeemToUnderlying[_setToken];

    for(uint256 i = 0; i < positionsLength; i++) {
        // Check that the given position is an equity position
        if(positions[i].unit > 0) {
            address component = positions[i].component;
            if(_isWrappedFCash(component)) {
                IWrappedfCashComplete fCashPosition = IWrappedfCashComplete(component);
                if(fCashPosition.hasMatured()) {
                    (IERC20 receiveToken,) = fCashPosition.getToken(toUnderlying);
                    if(address(receiveToken) == ETH_ADDRESS) {
                        receiveToken = weth;
                    }
                    uint256 fCashBalance = fCashPosition.balanceOf(address(_setToken));
                    _redeemFCashPosition(_setToken, fCashPosition, receiveToken, fCashBalance, 0);
                }
            }
        }
    }
}
```

## Impact

User will not be able to purchase or redeem the setToken. User's fund will stuck in the SetToken Contract. Unable to remove matured fCash positions from SetToken and update positions of its asset token.

## Recommended Mitigation Steps

This is a problem commonly encountered whenever a method of a smart contract calls another contract – you cannot rely on the other contract to work 100% of the time, and it is dangerous to assume that the external call will always be successful. 

It is recommended to:

- Consider alternate method of updating the asset position so that the SetToken's core functions (e.g. issuance and redemption) will not be locked if one of the matured fCash redemptions fails.

- Evaluate if `_redeemMaturedPositions` really need to be called during SetToken's issuance and redemption. If not, consider removing them from the hooks, so that any issue or revert within `_redeemMaturedPositions` won't cause the SetToken's issuance and redemption functions to stop working entirely.
- Consider implementing additional function to give manager/user an option to specify a list of matured fCash positions to redeem instead of forcing them to redeem all matured fCash positions at one go.

