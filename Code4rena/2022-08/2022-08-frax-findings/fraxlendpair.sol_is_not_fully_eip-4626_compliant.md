## Tags

- bug
- 2 (Med Risk)
- disagree with severity
- sponsor confirmed
- edited-by-warden

# [FraxlendPair.sol is not fully EIP-4626 compliant](https://github.com/code-423n4/2022-08-frax-findings/issues/79) 

# Lines of code

https://github.com/code-423n4/2022-08-frax/blob/c4189a3a98b38c8c962c5ea72f1a322fbc2ae45f/src/contracts/FraxlendPair.sol#L136-L138
https://github.com/code-423n4/2022-08-frax/blob/c4189a3a98b38c8c962c5ea72f1a322fbc2ae45f/src/contracts/FraxlendPair.sol#L140-L142


# Vulnerability details

## Impact
FraxlendPair.sol is not EIP-4626 compliant, variation from the standard could break composability and potentially lead to loss of funds

## Proof of Concept

According to EIP-4626 method specifications (https://eips.ethereum.org/EIPS/eip-4626)

For maxDeposit:

    MUST factor in both global and user-specific limits, like if deposits are entirely disabled (even temporarily) it MUST return 0.

For maxMint:

    MUST factor in both global and user-specific limits, like if mints are entirely disabled (even temporarily) it MUST return 0.

When FraxlendPair.sol is paused, deposit and mint are both disabled. This means that maxMint and maxDeposit should return 0 when the contract is paused. 

The current implementations of maxMint and maxDeposit do not follow this specification:

    function maxDeposit(address) external pure returns (uint256) {
        return type(uint128).max;
    }

    function maxMint(address) external pure returns (uint256) {
        return type(uint128).max;
    }

No matter the state of the contract they always return uint128.max, but they should return 0 when the contract is paused.

## Tools Used

## Recommended Mitigation Steps

maxDeposit and maxMint should be updated to return 0 when contract is paused. Use of the whenNotPaused modifier is not appropriate because that would cause a revert and maxDeposit and maxMint should never revert according to EIP-4626