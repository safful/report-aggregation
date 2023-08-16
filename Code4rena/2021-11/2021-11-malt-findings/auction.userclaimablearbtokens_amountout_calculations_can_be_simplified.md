## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Auction.userClaimableArbTokens amountOut calculations can be simplified](https://github.com/code-423n4/2021-11-malt-findings/issues/67) 

# Handle

hyh


# Vulnerability details

## Impact

Gas is overspent on access and operations.

## Proof of Concept

```amountOut``` is calculated in 3 steps, which can be made simpler.
https://github.com/code-423n4/2021-11-malt/blob/main/src/contracts/Auction.sol#L302-305

## Recommended Mitigation Steps

Now:
```
uint256 amountTokens = commitment.commitment.mul(auction.pegPrice).div(price);
uint256 redeemedTokens = commitment.redeemed.mul(auction.pegPrice).div(price);
uint256 amountOut = amountTokens.mul(claimablePerc).div(auction.pegPrice).sub(redeemedTokens);
```

To be (```amountTokens``` and ```redeemedTokens``` aren't used elsewhere):
```
 /*
* uint256 amountTokens = commitment.commitment.mul(auction.pegPrice).div(price);
* uint256 redeemedTokens = commitment.redeemed.mul(auction.pegPrice).div(price);
* uint256 amountOut = amountTokens.mul(claimablePerc).div(auction.pegPrice).sub(redeemedTokens);
*/
uint256 redeemed = commitment.redeemed.mul(auction.pegPrice);
uint256 amountOut = commitment.commitment.mul(claimablePerc).sub(redeemed).div(price);
```


