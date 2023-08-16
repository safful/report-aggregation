## Tags

- bug
- 1 (Low Risk)
- disagree with severity
- sponsor confirmed

# [Lost annualized fees due to early division](https://github.com/code-423n4/2021-12-amun-findings/issues/155) 

# Handle

kenzo


# Vulnerability details

An early division in `calcOutStandingAnnualizedFee` will lose accuracy of the fees calculation.

## Impact
Lost fees for the protocol.

## Proof of Concept
`calcOutStandingAnnualizedFee` uses the following formula to calculate the fees: [(Code ref)](https://github.com/code-423n4/2021-12-amun/blob/main/contracts/basket/contracts/facets/Basket/BasketFacet.sol#L259:#L262)
```
return
            totalSupply.mul(annualizedFee).div(10**18).mul(timePassed).div(
                365 days
            );
```

Since it is dividing by 10**18 in the middle of the calculation, the remainder of that division will be lost.
This is why in Solidity, division should come in the end.

## Recommended Mitigation Steps
Move `div(10**18)` to the end of the calculation.

By the way, you might wanna consider changing your fee structure to a smaller basis. Making the base 1e18 gives you granularity that you might not need. For example, if the annualized fee is planned to be 1%, instead of it being 1e15 out of 1e18 it can be 100 out of 10000. This will help prevent overflow if that was the reason you chose to divide in the midst of the calculation.

