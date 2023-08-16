## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# ["constants" expressions are expressions, not constants, so constant `keccak` variables results in extra hashing (and so gas).](https://github.com/code-423n4/2021-12-yetifinance-findings/issues/175) 

# Handle

Dravee


# Vulnerability details

## Impact
In a number of places a `keccak("string")` expression is assigned to a `constant` variable. Due to how `constant` variables are implemented this results in the hash being recomputed each time that the variable is used, spending the gas necessary to perform this action.

If these variables were to be `immutable` this hash is calculated once at deploy time and then the result is saved to be used directly at runtime rather than recalculating, saving the cost of hashing.

See: [ethereum/solidity#9232](https://github.com/ethereum/solidity/issues/9232)

## Proof of Concept
```
YETI\YETIToken.sol:
  50:     bytes32 private constant _PERMIT_TYPEHASH = keccak256("Permit(address owner,address spender,uint256 value,uint256 nonce,uint256 deadline)");
  51:     bytes32 private constant _TYPE_HASH = keccak256("EIP712Domain(string name,string version,uint256 chainId,address verifyingContract)");

YETI\BoringCrypto\Domain.sol:
  10:     bytes32 private constant DOMAIN_SEPARATOR_SIGNATURE_HASH = keccak256("EIP712Domain(uint256 chainId,address verifyingContract)");
```

## Tools Used
VS Code

## Recommended Mitigation Steps
Change all `constant` hashes to be `immutable`

