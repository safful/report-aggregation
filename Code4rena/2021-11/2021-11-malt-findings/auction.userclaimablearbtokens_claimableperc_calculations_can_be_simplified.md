## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Auction.userClaimableArbTokens claimablePerc calculations can be simplified](https://github.com/code-423n4/2021-11-malt-findings/issues/50) 

# Handle

hyh


# Vulnerability details

## Impact

Gas is overspent on access and operations.

## Proof of Concept

```claimablePerc``` is calculated in two steps, which can be made simpler.
https://github.com/code-423n4/2021-11-malt/blob/main/src/contracts/Auction.sol#L294

## Recommended Mitigation Steps

Now:
```
uint256 totalTokens = auction.commitments.mul(auction.pegPrice).div(auction.finalPrice);
uint256 claimablePerc = auction.claimableTokens.mul(auction.pegPrice).div(totalTokens);
```

To be (```totalTokens``` isn't used elsewhere):
```
uint256 claimablePerc = auction.claimableTokens.mul(auction.finalPrice).div(auction.commitments);
```

