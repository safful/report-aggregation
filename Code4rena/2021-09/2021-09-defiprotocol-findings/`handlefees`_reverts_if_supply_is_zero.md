## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [`handleFees` reverts if supply is zero](https://github.com/code-423n4/2021-09-defiprotocol-findings/issues/264) 

# Handle

0xsanson


# Vulnerability details

## Impact
In Basket.sol, `handleFees` computes the following:
`uint256 newIbRatio = ibRatio * startSupply / totalSupply()`.

In the case that `totalSupply() = 0` (every holder burned their basket), the function reverts since there's a 0/0. This issue won't let new people mint, since `handleFees` is called before any minting.

## Proof of Concept
https://github.com/code-423n4/2021-09-defiProtocol/blob/main/contracts/contracts/Basket.sol#L124

## Tools Used
editor

## Recommended Mitigation Steps
Consider adding a check before the division.
```
if (startSupply == 0) {
	return;
}
```

