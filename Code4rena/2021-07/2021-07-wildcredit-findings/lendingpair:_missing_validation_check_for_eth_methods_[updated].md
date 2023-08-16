## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed

# [LendingPair: Missing validation check for ETH methods [Updated]](https://github.com/code-423n4/2021-07-wildcredit-findings/issues/60) 

# Handle

greiart


# Vulnerability details

## Impact

The `depositRepayETH()`, `withdrawBorrowETH()`, `withdrawAllETH()` and `repayAllETH()` fail to check if ETH is an asset of the lending pair (ie. if ETH is either tokenA or tokenB).

From manually tracing the `depositRepayETH()` function, attempts to call it will revert in `_mintSupply()` when `lpToken[_token].mint(_account, _amount);` is called, since the `lpToken` is only initialized for only tokenA and tokenB.

Nevertheless, it is recommended to perform token validation for the ETH methods as well, since it should be treated like other ERC20 tokens. It also helps to avoid wasting gas.

## Referenced Codelines

[https://github.com/code-423n4/2021-07-wildcredit/blob/main/contracts/LendingPair.sol#L83](https://github.com/code-423n4/2021-07-wildcredit/blob/main/contracts/LendingPair.sol#L83)

[https://github.com/code-423n4/2021-07-wildcredit/blob/main/contracts/LendingPair.sol#L106](https://github.com/code-423n4/2021-07-wildcredit/blob/main/contracts/LendingPair.sol#L106)

[https://github.com/code-423n4/2021-07-wildcredit/blob/main/contracts/LendingPair.sol#L131](https://github.com/code-423n4/2021-07-wildcredit/blob/main/contracts/LendingPair.sol#L131)

[https://github.com/code-423n4/2021-07-wildcredit/blob/main/contracts/LendingPair.sol#L156](https://github.com/code-423n4/2021-07-wildcredit/blob/main/contracts/LendingPair.sol#L156)

## Recommended Mitigation Steps

Include `_validateToken()` in the equivalent ETH functions. An alternative suggestion is to drop the ETH methods (eg. combine `depositRepayETH()` into `depositRepay()` and doing a bit of refactoring to make native ETH deposits / withdrawals possible.

```jsx
// public constant ETH_ADDRESS = '';
// define a special constant address for ether != WETH address
function depositRepay(address _account, address _token, uint _amount) external payable {
    _validateTokenAndValue(_token, msg.value, _amount);
    accrueAccount(_account);

    _depositRepay(_account, _token, _amount);
		_handleDeposit{value:msg.value}(_token, _amount);
}

function _validateTokenAndValue(address _token, uint etherWei, uint amount) internal view {
		address token;
		if (_token == ETH_ADDRESS) {
			token = WETH;
			require(etherWei == amount, "LendingPair: invalid etherWei");
		} else {
			token = _token;
			require(etherWei == 0, "LendingPair: invalid etherWei");
		}
    require(token == tokenA || token == tokenB, "LendingPair: invalid token");
}

function _handleDeposit(address _token, uint _amount) internal payable {
	(_token == ETH_ADDRESS) ?
			WETH.deposit{ value: msg.value }() :
			_safeTransferFrom(_token, msg.sender, _amount);
}
```

