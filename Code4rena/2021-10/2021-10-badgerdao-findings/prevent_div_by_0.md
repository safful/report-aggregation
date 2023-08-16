## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [PREVENT DIV BY 0](https://github.com/code-423n4/2021-10-badgerdao-findings/issues/70) 

# Handle

defsec


# Vulnerability details

## Impact

On several locations in the code precautions are taken not to divide by 0, because this will revert the code. However on some locations this isn’t done.

Especially in the balanceToShares function div(pricePerShare) which isn’t checked. 

That will cause to revert on the transfer and transferFrom function. Oracle pricePerShare variable should be cheked on the balance calculation.

## Proof of Concept

1. Navigate to the following contracts,

"https://github.com/code-423n4/2021-10-badgerdao/blob/9d4734becebd729299f154c0cfa1d3a7f06cccfb/contracts/WrappedIbbtcEth.sol#L156"

"https://github.com/code-423n4/2021-10-badgerdao/blob/9d4734becebd729299f154c0cfa1d3a7f06cccfb/contracts/WrappedIbbtc.sol#L148"

2. If oracle fails, the pricePerShare variable will be equal to zero therefore div by zero will occur.

## Tools Used

Review

## Recommended Mitigation Steps

Recommend making sure division by 0 won’t occur by checking the variables beforehand and handling this edge case.

