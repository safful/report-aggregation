## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Failed transfer with low level call could be overlooked](https://github.com/code-423n4/2021-12-amun-findings/issues/78) 

# Handle

harleythedog


# Vulnerability details

## Impact
The `CallFacet.sol` contract has the function `_call` :
```
function  _call(
	address  _target,
	bytes  memory  _calldata,
	uint256  _value
) internal {
	require(address(this).balance >= _value, "ETH_BALANCE_TOO_LOW");
	(bool success, ) = _target.call{value: _value}(_calldata);
	require(success, "CALL_FAILED");
	emit  Call(msg.sender, _target, _calldata, _value);
}
```
This function is utilized in a lot of different places. According to the [Solidity docs]([https://docs.soliditylang.org/en/develop/control-structures.html#error-handling-assert-require-revert-and-exceptions](https://docs.soliditylang.org/en/develop/control-structures.html#error-handling-assert-require-revert-and-exceptions)), "The low-level functions `call`, `delegatecall` and `staticcall` return `true` as their first return value if the account called is non-existent, as part of the design of the EVM. Account existence must be checked prior to calling if needed".  
  
As a result, it is possible that this call will not work but `_call` will not notice anything went wrong. It could be possible that a user is interacting with an exchange or token that has been deleted, but `_call` will not notice that something has gone wrong and as a result, ether can become stuck in the contract. For this reason, it would be better to also check for the contract's existence prior to executing `_target.call`. 

For reference, see a similar high severity reported in a Uniswap audit here (report # 9): https://github.com/Uniswap/v3-core/blob/main/audits/tob/audit.pdf



## Proof of Concept
See `_call` here: https://github.com/code-423n4/2021-12-amun/blob/98f6e2ff91f5fcebc0489f5871183566feaec307/contracts/basket/contracts/facets/Call/CallFacet.sol#L108.

## Tools Used
Inspection

## Recommended Mitigation Steps
To ensure tokens don't get stuck in edge case where user is interacting with a deleted contract, make sure to check that contract actually exists before calling it. 

