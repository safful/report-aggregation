## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [execute in VoteProxy should be payable](https://github.com/code-423n4/2022-02-concur-findings/issues/17) 

# Handle

wuwe1


# Vulnerability details

## Impact
`execute` will revert when `msg.value > 0`

## Proof of Concept
Lacking `payable` mutability specifier.

https://github.com/code-423n4/2022-02-concur/blob/main/contracts/VoteProxy.sol#L28-L35

```solidity
    function execute(
        address _to,
        uint256 _value,
        bytes calldata _data
    ) external onlyOwner returns (bool, bytes memory) {
        (bool success, bytes memory result) = _to.call{value: _value}(_data);
        return (success, result);
    }
```


## Recommended Mitigation Steps

Add `payable` mutability specifier.

