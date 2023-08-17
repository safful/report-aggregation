## Tags

- bug
- QA (Quality Assurance)
- sponsor confirmed

# [Highly Unsafe Pattern of Minting the Additional Reward Tokens at `VeAssetDepositor.sol`](https://github.com/code-423n4/2022-05-vetoken-findings/issues/29) 

# Lines of code

https://github.com/code-423n4/2022-05-vetoken/blob/2d7cd1f6780a9bcc8387dea8fecfbd758462c152/contracts/VeAssetDepositor.sol#L117-L120


# Vulnerability details

## Impact 

In [VeAssetDepositor.sol#L117-L120](https://github.com/code-423n4/2022-05-vetoken/blob/2d7cd1f6780a9bcc8387dea8fecfbd758462c152/contracts/VeAssetDepositor.sol#L117-L120), the condition to mint the additional rewards tokens to the user is `if (incentiveVeAsset > 0)`. However, the `incentiveVeAsset` variable is only updated to zero after an external call to the `ITokenMinter` contract. This lacks the Checks Effects and Interactions safety pattern. In the event that the **wrong** minter contract has been initialised, an attacker could potentially drain all the additional reward tokens via a reentrancy attack.

## Proof of Concept

- <https://fravoll.github.io/solidity-patterns/checks_effects_interactions.html>

## Recommended Mitigation Steps

Be sure to follow the [Checks Effects and Interactions safety pattern](https://fravoll.github.io/solidity-patterns/checks_effects_interactions.html) and update the `incentiveVeAsset = 0` before minting the token for the user. Alternatively, the developers can also add the `nonReentrant()` [modifier](https://docs.openzeppelin.com/contracts/4.x/api/security#ReentrancyGuard) from OpenZeppelin to prevent any sort of potential reentrancy attacks.

