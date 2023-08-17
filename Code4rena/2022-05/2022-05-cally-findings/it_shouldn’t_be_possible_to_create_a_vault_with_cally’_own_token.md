## Tags

- bug
- 2 (Med Risk)
- disagree with severity
- resolved
- sponsor confirmed

# [It shouldn’t be possible to create a vault with Cally’ own token](https://github.com/code-423n4/2022-05-cally-findings/issues/224) 

# Lines of code

https://github.com/code-423n4/2022-05-cally/blob/main/contracts/src/Cally.sol#L193
https://github.com/code-423n4/2022-05-cally/blob/main/contracts/src/Cally.sol#L199


# Vulnerability details

## Impact

Affected code:

- [https://github.com/code-423n4/2022-05-cally/blob/main/contracts/src/Cally.sol#L193](https://github.com/code-423n4/2022-05-cally/blob/main/contracts/src/Cally.sol#L193)
- [https://github.com/code-423n4/2022-05-cally/blob/main/contracts/src/Cally.sol#L199](https://github.com/code-423n4/2022-05-cally/blob/main/contracts/src/Cally.sol#L199)

Currently it’s possible to create an ERC-721 vault using Cally’ own address as `token`, and using the freshly minted vault id as `tokenIdOrAmount`. This results in a new vault whose ownership is passed to Cally contract immediately upon creation.

The vault allows users to perform `buyOption` and increase the ETH balance of the Cally contract itself, which is still the vault beneficiary. As soon as an user calls `exercise`, she will receive the `vault.tokenIdOrAmount` in exchange, which in this case coincides with the vault nft. However this is of no good because the final user may just initiate a withdrawal, which will:

- always fail because the vault id is burned ([https://github.com/code-423n4/2022-05-cally/blob/main/contracts/src/Cally.sol#L335](https://github.com/code-423n4/2022-05-cally/blob/main/contracts/src/Cally.sol#L335)) and then transferred back to the user ([https://github.com/code-423n4/2022-05-cally/blob/main/contracts/src/Cally.sol#L344](https://github.com/code-423n4/2022-05-cally/blob/main/contracts/src/Cally.sol#L344))
- leave all the ETH unredemable in Cally contract

So the vault will be unusable and the ETH deposited by users to buy/exercise options will remain locked in Cally contract

## Proof of Concept

- Current vault id is, let’s say, 11
- User deploys a vault with Cally’ address as `token` and `13` as `tokenIdOrAmount`
- Since `createVault()` mints the vault token to the user, and then transfers the underlying address from the user, an user is able to create a vault with something she doesn’t own at the moment of the `createVault()` function call, because it’s created while the function runs
- The vault `13` is pretty limited in functionality, because Cally’ smart contract is the owner
- However, users can still buy options: so Alice and Bob deposit their premiums
- Whoever `exercise` the active option, becomes the vault owner now; this is of no good because no one can actually call `withdraw()` as it will always revert, and no one can recover the ETH deposited by Alice and Bob as they are locked forever

## Tools Used

Editor

## Recommended Mitigation Steps

Add the following check at the start of `createVault()`:

```jsx
require(token != address(this), "Cant use Cally as token");
```

