## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Users can avoid paying borrowing interest after the fyToken matures](https://github.com/code-423n4/2021-05-yield-findings/issues/71) 

# Handle

shw


# Vulnerability details

## Impact

According to the protocol design, users have to pay borrowing interest when repaying the debt with underlying tokens after maturity. However, a user can give his vault to `Witch` and then buy all his collateral using underlying tokens to avoid paying the interest. Besides, this bug could make users less incentivized to repay the debt before maturity and hold the underlying tokens until liquidation.

## Proof of Concept

1. A user creates a new vault and opens a borrowing position as usual.
2. The maturity date passed. If the user wants to close the position using underlying tokens, he has to pay a borrowing interest (line 350 in `Ladle`), which is his debt multiplied by the rate accrual (line 373).
3. Now, the user wants to avoid paying the borrowing interest. He gives his vault to `Witch` by calling the function `batch` of `Ladle` with the operation `GIVE`.
4. He then calls the function `buy` of `Witch` with the corresponding `vaultId` to buy all his collateral using underlying tokens.

In the last step, the `elapsed` time (line 61) is equal to the current timestamp since the vault is never grabbed by `Witch` before, and thus the auction time of the vault, `cauldron.auctions(vaultId)`, is 0 (the default mapping value). Therefore, the collateral is sold at a price of `balances_.art/balances_.ink` (line 74). The user can buy `balances_.ink` amount of collateral using `balances_.art` but not paying for borrowing fees.

Referenced code:
[Ladle.sol#L350](https://github.com/code-423n4/2021-05-yield/blob/main/contracts/Ladle.sol#L350)
[Ladle.sol#L368-L377](https://github.com/code-423n4/2021-05-yield/blob/main/contracts/Ladle.sol#L368-L377)
[Ladle.sol#L267-L272](https://github.com/code-423n4/2021-05-yield/blob/main/contracts/Ladle.sol#L267-L272)
[Cauldron.sol#L234-L252](https://github.com/code-423n4/2021-05-yield/blob/main/contracts/Cauldron.sol#L234-L252)
[Witch.sol#L61](https://github.com/code-423n4/2021-05-yield/blob/main/contracts/Witch.sol#L61)
[Witch.sol#L74](https://github.com/code-423n4/2021-05-yield/blob/main/contracts/Witch.sol#L74)

## Recommended Mitigation Steps

Do not allow users to `give` vaults to `Witch`. To be more careful, require `vaultOwners[vaultId]` and `cauldron.auctions(vaultId)` to be non-zero at the beginning of function `buy`.

