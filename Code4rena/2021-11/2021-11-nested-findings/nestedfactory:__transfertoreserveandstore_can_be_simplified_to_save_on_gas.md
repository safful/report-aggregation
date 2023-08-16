## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [NestedFactory: _transferToReserveAndStore can be simplified to save on gas](https://github.com/code-423n4/2021-11-nested-findings/issues/5) 

# Handle

PierrickGT


# Vulnerability details

## Impact
In [_transferToReserveAndStore](https://github.com/code-423n4/2021-11-nested/blob/cbd39fe7d76ed8c84eb767a5f3b6eba83e034656/contracts/NestedFactory.sol#L426), `_token` is casted 3 times to `IERC20` and `reserve` is loaded and casted to `address` 4 times. We can simplify the function and save on gas.

## Proof of Concept
`_token` should be passed to the function as an `IERC20`, this way we avoid to cast it 3 times. `reserve` should be stored in a variable to avoid 3 unnecessary sloads and casting.

## Recommended Mitigation Steps
The following changes are recommended.

```
    function _transferToReserveAndStore(
        IERC20 _token,
        uint256 _amount,
        uint256 _nftId
    ) private {
        address reserveAddress = address(reserve);
        uint256 balanceReserveBefore = _token.balanceOf(reserveAddress);

        // Send output to reserve
        _token.safeTransfer(reserveAddress, _amount);

        uint256 balanceReserveAfter = _token.balanceOf(reserveAddress);

        nestedRecords.store(_nftId, address(_token), balanceReserveAfter - balanceReserveBefore, reserveAddress);
    }
```

After this subsequent change, `_outputToken` will need to be casted to `IERC20` on [L386](https://github.com/code-423n4/2021-11-nested/blob/cbd39fe7d76ed8c84eb767a5f3b6eba83e034656/contracts/NestedFactory.sol#L386).

`_transferToReserveAndStore(IERC20(_outputToken), amounts[0], _nftId);`

And no need to cast `_outputToken` anymore on [L357](https://github.com/code-423n4/2021-11-nested/blob/cbd39fe7d76ed8c84eb767a5f3b6eba83e034656/contracts/NestedFactory.sol#L357).

`_transferToReserveAndStore(_outputToken, amountBought - feesAmount, _nftId);`

