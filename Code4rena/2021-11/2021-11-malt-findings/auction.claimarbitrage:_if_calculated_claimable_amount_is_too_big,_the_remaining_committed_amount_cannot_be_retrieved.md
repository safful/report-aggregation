## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [Auction.claimArbitrage: if calculated claimable amount is too big, the remaining committed amount cannot be retrieved](https://github.com/code-423n4/2021-11-malt-findings/issues/353) 

# Handle

hyh


# Vulnerability details

## Impact

A condition requires that calculated retrievable amount shouldn't be too big. If it is the function fails and the remaining portion of commitment is frozen.

As the amount is calculated by the system a user cannot do anything to retrieve remaining part of commitment, if any.

## Proof of Concept

```claimArbitrage``` fails if calculated redemption is higher than remaining commitment:
https://github.com/code-423n4/2021-11-malt/blob/main/src/contracts/Auction.sol#L230

```userClaimableArbTokens``` calculated amount can be bigger than remaining user funds:
https://github.com/code-423n4/2021-11-malt/blob/main/src/contracts/Auction.sol#L279

## Recommended Mitigation Steps

If the freezing of remainder amount is not intentional then substitute require with ceiling the amount to be retrieved with the remaining part.

Now:
```
require(redemption <= remaining.add(1), "Cannot claim more tokens than available");
```

To be:
```
if (redemption > remaining) {
	redemption = remaining;
}
```


