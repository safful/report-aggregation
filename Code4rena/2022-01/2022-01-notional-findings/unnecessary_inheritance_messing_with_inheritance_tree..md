## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Unnecessary inheritance messing with inheritance tree.](https://github.com/code-423n4/2022-01-notional-findings/issues/62) 

# Handle

TomFrenchBlockchain


# Vulnerability details

## Impact
Extra boilerplate code (including the whole of the _afterTokenTransfer function)

## Proof of Concept

sNOTE inherits from both ERC20Upgradeable and ERC20VotesUpgradeable.

https://github.com/code-423n4/2022-01-notional/blob/d171cad9e86e0d02e0909eb66d4c24ab6ea6b982/contracts/sNOTE.sol#L15

This causes us to have to add explicit helpers for how to handle the inheritance tree to a bunch of functions

https://github.com/code-423n4/2022-01-notional/blob/d171cad9e86e0d02e0909eb66d4c24ab6ea6b982/contracts/sNOTE.sol#L315

https://github.com/code-423n4/2022-01-notional/blob/d171cad9e86e0d02e0909eb66d4c24ab6ea6b982/contracts/sNOTE.sol#L328

https://github.com/code-423n4/2022-01-notional/blob/d171cad9e86e0d02e0909eb66d4c24ab6ea6b982/contracts/sNOTE.sol#L362

https://github.com/code-423n4/2022-01-notional/blob/d171cad9e86e0d02e0909eb66d4c24ab6ea6b982/contracts/sNOTE.sol#L381

If we look at OZ however, we can see that ERC20VotesUpgradeable inherits from ERC20PermitUpgradeable which in turn inherits from ERC20Upgradeable 

https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/blob/fd165faaf00587377b5ab93be3cafb4ffdc96976/contracts/token/ERC20/extensions/ERC20VotesUpgradeable.sol#L28

https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/blob/fd165faaf00587377b5ab93be3cafb4ffdc96976/contracts/token/ERC20/extensions/draft-ERC20PermitUpgradeable.sol#L23

There's then no real reason for sNOTE to inherit from ERC20Upgradeable directly. Removing this inheritance should allow you to remove a bunch of the explicit overrides you have.

## Recommended Mitigation Steps

Remove direct inheritance of ERC20Upgradeable and remove all the `override(ERC20Upgradeable, ERC20VotesUpgradeable)` stuff. You should be able to just delete `_afterTokenTransfer` in its entirety.

