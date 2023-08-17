## Tags

- bug
- 2 (Med Risk)
- resolved
- sponsor confirmed

# [User's may accidentally overpay in `buyOption()` and the excess will be paid to the vault creator](https://github.com/code-423n4/2022-05-cally-findings/issues/84) 

# Lines of code

https://github.com/code-423n4/2022-05-cally/blob/1849f9ee12434038aa80753266ce6a2f2b082c59/contracts/src/Cally.sol#L223-L224


# Vulnerability details

## Impact

It is possible for a user purchasing an option to accidentally overpay the premium during `buyOption()`.

Any excess funds paid for in excess of the premium will be transferred to the vault creator.

The premium is fixed at the time the vault is first created by `vault.premiumIndex`. Hence there is no need to allow users to overpay since there will be no benefit. 

## Proof of Concept

`buyOption()` allows `msg.value > premium`

```solidity
        uint256 premium = getPremium(vaultId);
        require(msg.value >= premium, "Incorrect ETH amount sent");
```

## Recommended Mitigation Steps

Consider modifying the check such that the `msg.value` is exactly equal to the `premuim`. e.g.

```solidity
        uint256 premium = getPremium(vaultId);
        require(msg.value == premium, "Incorrect ETH amount sent");
```

