## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [Transferring any amount of the underlying token to the CNote contract will make the contract functions unusable](https://github.com/code-423n4/2022-06-canto-findings/issues/227) 

# Lines of code

https://github.com/Plex-Engineer/lending-market/blob/755424c1f9ab3f9f0408443e6606f94e4f08a990/contracts/CNote.sol#L43
https://github.com/Plex-Engineer/lending-market/blob/755424c1f9ab3f9f0408443e6606f94e4f08a990/contracts/CNote.sol#L114
https://github.com/Plex-Engineer/lending-market/blob/755424c1f9ab3f9f0408443e6606f94e4f08a990/contracts/CNote.sol#L198
https://github.com/Plex-Engineer/lending-market/blob/755424c1f9ab3f9f0408443e6606f94e4f08a990/contracts/CNote.sol#L310


# Vulnerability details

## Impact

The contract expects the balance of the underlying token to == 0 at all points when calling the contract functions by requiring getCashPrior() == 0, which checks token.balanceOf(address(this)) where token is the underlying asset.

An attacker can transfer any amount of the underlying asset directly to the contract and make all of the functions requiring getCashPrior() == 0 to revert. 

## Proof of Concept
[CNote.sol#L43](https://github.com/Plex-Engineer/lending-market/blob/755424c1f9ab3f9f0408443e6606f94e4f08a990/contracts/CNote.sol#L43)
[CNote.sol#L114](https://github.com/Plex-Engineer/lending-market/blob/755424c1f9ab3f9f0408443e6606f94e4f08a990/contracts/CNote.sol#L114)
[CNote.sol#198](https://github.com/Plex-Engineer/lending-market/blob/755424c1f9ab3f9f0408443e6606f94e4f08a990/contracts/CNote.sol#L198)
[CNote.sol#310](https://github.com/Plex-Engineer/lending-market/blob/755424c1f9ab3f9f0408443e6606f94e4f08a990/contracts/CNote.sol#L310)

1. Attacker gets any balance of Note (amount = 1 token)
2. Attacker transfers the token to CNote which uses Note as an underlying asset, by calling note.transfer(CNoteAddress, amount). The function is available since Note inherits from ERC20
3. Any calls to CNote functions now revert due to getCashPrior() not being equal to 0

## Recommended Mitigation Steps
Instead of checking the underlying token balance via balanceOf(address(this)) the contract could hold an internal balance of the token, mitigating the impact of tokens being forcefully transferred to the contract.

