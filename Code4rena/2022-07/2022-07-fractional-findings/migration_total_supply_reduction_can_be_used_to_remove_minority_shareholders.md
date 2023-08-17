## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed
- old-submission-method

# [Migration total supply reduction can be used to remove minority shareholders](https://github.com/code-423n4/2022-07-fractional-findings/issues/612) 

# Lines of code

https://github.com/code-423n4/2022-07-fractional/blob/8f2697ae727c60c93ea47276f8fa128369abfe51/src/modules/Migration.sol#L469-L472
https://github.com/code-423n4/2022-07-fractional/blob/8f2697ae727c60c93ea47276f8fa128369abfe51/src/modules/Migration.sol#L95-L98


# Vulnerability details

As new total supply can be arbitrary, setting it significantly lower than current (say to 100 when it was 1e9 before) can be used to remove current minority shareholders, whose shares will end up being zero on a precision loss due to low new total supply value. This can go unnoticed as the effect is implementation based.

During Buyout the remaining shareholders are left with ETH funds based valuation and can sell the shares, but the minority shareholders that did contributed to the Migration, that could have other details favourable to them, may not realize that new shares will be calculated with the numerical truncation as a result of the new total supply introduction.

Setting the severity to medium as this is a fund loss impact conditional on a user not understanding the particulars of the implementation.

## Proof of Concept

Currently migrateFractions() calculates new shares to be transferred for a user as a fraction of her contribution:

https://github.com/code-423n4/2022-07-fractional/blob/8f2697ae727c60c93ea47276f8fa128369abfe51/src/modules/Migration.sol#L469-L472

```solidity
        // Calculates share amount of fractions for the new vault based on the new total supply
        uint256 newTotalSupply = IVaultRegistry(registry).totalSupply(newVault);
        uint256 shareAmount = (balanceContributedInEth * newTotalSupply) /
            totalInEth;
```

If Bob the msg.sender is a minority shareholder who contributed to Migration with say some technical enhancements of the Vault, not paying attention to the total supply reduction, his share can be lost on commit():

https://github.com/code-423n4/2022-07-fractional/blob/8f2697ae727c60c93ea47276f8fa128369abfe51/src/modules/Migration.sol#L209-L210

```solidity
            // Starts the buyout process
            IBuyout(buyout).start{value: proposal.totalEth}(_vault);
```

As commit() starts the Buyout, Bob will not be able to withdraw as both leave() and withdrawContribution() require INACTIVE state:

https://github.com/code-423n4/2022-07-fractional/blob/8f2697ae727c60c93ea47276f8fa128369abfe51/src/modules/Migration.sol#L149-L150

```solidity
        State required = State.INACTIVE;
        if (current != required) revert IBuyout.InvalidState(required, current);
```

If Buyout be successful, Bob's share can be calculated as zero given his small initial share and reduction in the Vault total shares.

For example, if Bob's share together with the ETH funds he provided to Migration were cumulatively less than 1%, and new total supply is 100, he will lose all his contribution on commit() as migrateFractions() will send him nothing.

## Recommended Mitigation Steps

Consider requiring that the new total supply should be greater than the old one:

https://github.com/code-423n4/2022-07-fractional/blob/8f2697ae727c60c93ea47276f8fa128369abfe51/src/modules/Migration.sol#L95-L98

```solidity
        proposal.oldFractionSupply = IVaultRegistry(registry).totalSupply(
            _vault
        );
        proposal.newFractionSupply = _newFractionSupply;
+       require(proposal.newFractionSupply > proposal.oldFractionSupply, ""); // reference version
```

