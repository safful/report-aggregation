## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Wrong DOMAIN_SEPARATOR](https://github.com/code-423n4/2022-05-rubicon-findings/issues/38) 

# Lines of code

https://github.com/code-423n4/2022-05-rubicon/blob/521d50b22b41b1f52ff9a67ea68ed8012c618da9/contracts/rubiconPools/BathToken.sol#L199-L210


# Vulnerability details

## Impact
The `DOMAIN_SEPARATOR` is wrong calculated.

## Proof of Concept

In the `initialize` method of the `BathToken` contract, the `name` of the contract is used to calculate the `DOMAIN_SEPARATOR`, however said name is set later, so it will use an incorrect `name`, making it impossible to calculate the `DOMAIN_SEPARATOR` correctly.

```javascript
DOMAIN_SEPARATOR = keccak256(
    abi.encode(
        keccak256(
            "EIP712Domain(string name,string version,uint256 chainId,address verifyingContract)"
        ),
        keccak256(bytes(name)),
        keccak256(bytes("1")),
        chainId,
        address(this)
    )
);
name = string(abi.encodePacked(_symbol, (" v1")));
```

Affected source code:
- [BathToken.sol#L199-L210](https://github.com/code-423n4/2022-05-rubicon/blob/521d50b22b41b1f52ff9a67ea68ed8012c618da9/contracts/rubiconPools/BathToken.sol#L199-L210)

## Recommended Mitigation Steps
- Set the `name` before use it.

