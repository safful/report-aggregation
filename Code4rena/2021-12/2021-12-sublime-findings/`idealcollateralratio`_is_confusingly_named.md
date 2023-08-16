## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed

# [`idealCollateralRatio` is confusingly named](https://github.com/code-423n4/2021-12-sublime-findings/issues/79) 

# Handle

TomFrenchBlockchain


# Vulnerability details

## Impact
Weird naming
## Proof of Concept

Credit lines have an "ideal collateral ratio" which acts very much like a minimum collateral ratio, i.e. if you fall below it you can be liquidated.

https://github.com/code-423n4/2021-12-sublime/blob/9df1b7c4247f8631647c7627a8da9bdc16db8b11/contracts/CreditLine/CreditLine.sol#L1002

"Ideal" to me implies that you'll be hovering around that value sometimes drifting above and sometimes below but the systems drives the value back to the ideal so this is quite confusingly named imo.

## Recommended Mitigation Steps

Change `idealCollateralRatio` to `minCollateralRatio`

