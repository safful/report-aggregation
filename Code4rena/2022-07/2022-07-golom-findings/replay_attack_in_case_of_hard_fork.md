## Tags

- bug
- 2 (Med Risk)
- disagree with severity
- resolved
- sponsor confirmed
- selected-for-report

# [Replay attack in case of Hard fork](https://github.com/code-423n4/2022-07-golom-findings/issues/391) 

# Lines of code

https://github.com/code-423n4/2022-07-golom/blob/7bbb55fca61e6bae29e57133c1e45806cbb17aa4/contracts/core/GolomTrader.sol#L98


# Vulnerability details

## Impact
If there is ever a hardfork for Golom then EIP712_DOMAIN_TYPEHASH value will become invalid. This is because the chainId parameter is computed in constructor. This means even after hard fork chainId would remain same which is incorrect and could cause possible replay attacks

## Proof of Concept
1. Observe the constructor

```
constructor(address _governance) {
        // sets governance as owner
        _transferOwnership(_governance);

        uint256 chainId;
        assembly {
            chainId := chainid()
        }

        EIP712_DOMAIN_TYPEHASH = keccak256(
            abi.encode(
                keccak256('EIP712Domain(string name,string version,uint256 chainId,address verifyingContract)'),
                keccak256(bytes('GOLOM.IO')),
                keccak256(bytes('1')),
                chainId,
                address(this)
            )
        );
    }
```

2. As we can see the chainId is derived and then hardcoded in EIP712_DOMAIN_TYPEHASH 

3. This means even after hard fork, EIP712_DOMAIN_TYPEHASH value will remain same and point to incorrect chainId

## Recommended Mitigation Steps
The EIP712_DOMAIN_TYPEHASH variable should be recomputed everytime by placing current value of chainId