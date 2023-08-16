## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed

# [Magic number 30 could be a constant](https://github.com/code-423n4/2021-12-sublime-findings/issues/82) 

# Handle

sirhashalot


# Vulnerability details

## Impact

There are many instances of the value 30, usually used for exponents of base 10. Currently this integer value is used directly without a clear indication that this value relates to the decimals value, which could lead to one of these values being modified but not the other (perhaps by a typo), which is the basis for many past hacks. Coding best practices suggests using a constant integer to store this value in a way that clearly explains the purpose of this value to prevent confusion.

## Proof of Concept
The magic number 30 is found in dozens of places, including:

Pool/Repayments.sol
https://github.com/code-423n4/2021-12-sublime/blob/9df1b7c4247f8631647c7627a8da9bdc16db8b11/contracts/Pool/Repayments.sol#L164-L165 
https://github.com/code-423n4/2021-12-sublime/blob/9df1b7c4247f8631647c7627a8da9bdc16db8b11/contracts/Pool/Repayments.sol#L190

PriceOracle.sol file
https://github.com/code-423n4/2021-12-sublime/blob/9df1b7c4247f8631647c7627a8da9bdc16db8b11/contracts/PriceOracle.sol#L130-L131 
https://github.com/code-423n4/2021-12-sublime/blob/9df1b7c4247f8631647c7627a8da9bdc16db8b11/contracts/PriceOracle.sol#L88-L93 

## Tools Used

Manual review

## Recommended Mitigation Steps

Replace the magic number 30 with a variable explaining the meaning of this value, such as:
`uint8 private constant DECIMALS_EXPONENT = 30;`

