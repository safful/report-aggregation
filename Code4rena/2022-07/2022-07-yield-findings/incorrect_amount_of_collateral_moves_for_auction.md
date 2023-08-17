## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [Incorrect amount of Collateral moves for Auction](https://github.com/code-423n4/2022-07-yield-findings/issues/123) 

# Lines of code

https://github.com/code-423n4/2022-07-yield/blob/main/contracts/Witch.sol#L232


# Vulnerability details

## Impact
It was observed that the debt and collateral which moves for Auction is calculated incorrectly. In case where line.proportion is set to small value, chances are art will become lower than min debt. This causes whole collateral to go for auction, which was not expected

___

## Proof of Concept
1. Assume line.proportion is set to 10% which is a [valid value](https://github.com/code-423n4/2022-07-yield/blob/main/contracts/Witch.sol#L108)

2. Auction is started on Vault associated with collateral & base representing line from Step 1

3. Now debt and collateral to be sold are calculated in [_calcAuction](https://github.com/code-423n4/2022-07-yield/blob/main/contracts/Witch.sol#L223)

```
uint128 art = uint256(balances.art).wmul(line.proportion).u128();
        if (art < debt.min * (10**debt.dec)) art = balances.art;
        uint128 ink = (art == balances.art)
            ? balances.ink
            : uint256(balances.ink).wmul(line.proportion).u128();
```

4. Now lets say **debt (art)** on this vault was **amount 10**, **collateral (ink)** was **amount 9**, debt.min * (10**debt.dec) was **amount 2**

5. Below calculation occurs

```
uint128 art = uint256(balances.art).wmul(line.proportion).u128(); // which makes art = 10*10% =1
        if (art < debt.min * (10**debt.dec)) art = balances.art;   // since 1<2 so art=10
        uint128 ink = (art == balances.art)                                 // Since art is 10 so ink=9
            ? balances.ink
            : uint256(balances.ink).wmul(line.proportion).u128();
```

6. So full collateral and full debt are placed for Auction even though only 10% was meant for Auction. Even if it was lower than min debt, auction amount should have only increased upto the point where minimum debt limit is reached

___

## Recommended Mitigation Steps
Revise the calculation like below

```
uint128 art = uint256(balances.art).wmul(line.proportion).u128();
uint128 ink=0;
        if (art < debt.min * (10**debt.dec)) 
{
art = debt.min * (10**debt.dec);
(balances.ink<art) ? (ink=balances.ink) : (ink=art)
} else {
ink=uint256(balances.ink).wmul(line.proportion).u128();
}
```

