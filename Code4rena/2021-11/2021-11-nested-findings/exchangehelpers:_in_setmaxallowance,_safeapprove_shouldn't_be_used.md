## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [ExchangeHelpers: in setMaxAllowance, safeApprove shouldn't be used](https://github.com/code-423n4/2021-11-nested-findings/issues/50) 

# Handle

PierrickGT


# Vulnerability details

## Impact
In [setMaxAllowance](https://github.com/code-423n4/2021-11-nested/blob/cbd39fe7d76ed8c84eb767a5f3b6eba83e034656/contracts/libraries/ExchangeHelpers.sol#L34), `safeApprove` is used to increase allowance. As stated in the following Pull Request, `safeApprove` has been deprecated in favor of `safeIncreaseAllowance` and `safeDecreaseAllowance`.

https://github.com/OpenZeppelin/openzeppelin-contracts/pull/2268/files

This is because `safeApprove` shouldn't check for allowance, as explained in the issue below:

https://github.com/OpenZeppelin/openzeppelin-contracts/issues/2219

`approve` is actually vulnerable to a sandwich attack as explained in the following document and this check for allowance doesn't actually avoid it.

https://docs.google.com/document/d/1YLPtQxZu1UAvO9cZ1O2RPXBbT0mooh4DYKjA_jp-RLM/edit

## Proof of Concept
`safeIncreaseAllowance` should be used to increase allowance and `safeDecreaseAllowance` to decrease allowance to 0. We can also gain in code clarity by refactoring the `if else` statement and calling `_token.safeIncreaseAllowance(_spender, type(uint256).max);` only once.

## Recommended Mitigation Steps
The following changes are recommended.

```
function setMaxAllowance(IERC20 _token, address _spender) internal {
    uint256 _currentAllowance = _token.allowance(address(this), _spender);

    if (_currentAllowance != type(uint256).max) {
        // Decrease to 0 first for tokens mitigating the race condition
        _token.safeDecreaseAllowance(_spender, _currentAllowance);
    }

    _token.safeIncreaseAllowance(_spender, type(uint256).max);
}
```

