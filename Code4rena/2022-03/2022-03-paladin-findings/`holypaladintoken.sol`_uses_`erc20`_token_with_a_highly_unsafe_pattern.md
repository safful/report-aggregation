## Tags

- bug
- 2 (Med Risk)
- resolved
- sponsor confirmed

# [`HolyPaladinToken.sol` uses `ERC20` token with a highly unsafe pattern](https://github.com/code-423n4/2022-03-paladin-findings/issues/3) 

# Lines of code

https://github.com/code-423n4/2022-03-paladin/blob/main/contracts/open-zeppelin/ERC20.sol#L149


# Vulnerability details

## Impact
In `HolyPaladinToken.sol` it imports `ERC20.sol` with some changes from the original Open Zeppelin standard.  One change is that the `transferFrom()` function does not follow the Checks Effect and Interactions safety pattern to safely make external calls to other contracts. All checks should be handled first, then any effects/state updates,  followed by the external call to prevent reentrancy attacks.  Currently the `transferFrom()` function in `ERC20.sol` used by `HolyPaladinToken.sol` calls `_transfer()` first and then updates the `sender` allowance which is highly unsafe.  The openZeppelin `ER20.sol` contract which is the industry standard first updates the `sender` allowance before calling `_transfer`.  The external call should always be done last to avoid any double spending bugs or reentrancy attacks. 

## Proof of Concept
https://fravoll.github.io/solidity-patterns/checks_effects_interactions.html

https://github.com/code-423n4/2022-03-paladin/blob/main/contracts/open-zeppelin/ERC20.sol#L149

Open Zeppelins Implementation
https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC20/ERC20.sol

## Tools Used
Manual code review 

## Recommended Mitigation Steps
Be sure to follow the Checks Effects and Interactions safety pattern as the `transferFrom` function is one of the most important functions in any protocol.  Consider importing the Open Zeppelin `ERC20.sol` contract code directly as it is battle tested and safe code. 

