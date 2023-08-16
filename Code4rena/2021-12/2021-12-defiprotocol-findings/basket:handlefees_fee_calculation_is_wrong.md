## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Basket:handleFees fee calculation is wrong](https://github.com/code-423n4/2021-12-defiprotocol-findings/issues/43) 

# Handle

GiveMeTestEther


# Vulnerability details

## Impact
The fee calculation on L141 is wrong. It should only get divided by BASE and not (BASE - feePct)

## Proof of Concept
This shows dividing only by BASE is correct:
Assumptions: 
- BASE is 1e18 accordign to the code
- timeDiff is exactly ONE_YEAR (for easier calculations)
- startSupply is 1e18 (exactly one basket token, also represents 100% in fee terms)
- licenseFee is 1e15 (0.1%)
If we calculate the fee of one whole year and startSupply is one token (1e18, equal to 100%), the fee should be exactly the licenseFee  (1e15, 0.1%),

uint256 timeDiff = ONE_YEAR; 
uint256 feePct = timeDiff * licenseFee / ONE_YEAR;
=> therefore we have: feePct = licenseFee which is 1e15 (0.1%) according to our assumptions
uint256 fee = startSupply * feePct / BASE; // only divide by BASE
=> insert values => fee = 1e18 * licenseFee  / 1e18 = licenseFee  

This shows the math is wrong:

Assumptions: 
- BASE is 1e18 according to the code
- timeDiff is exactly ONE_YEAR (for easier calculations)
- startSupply is 1e18 (exactly one basket token, also represents 100% in fee terms)
- licenseFee is 1e15 (0.1%)
If we calculate the fee of one whole year and startSupply is one token (1e18, equal to 100%), the fee should be exactly the licenseFee  (1e15, 0.1%), but the fee is bigger than that.


uint256 timeDiff = ONE_YEAR; 
uint256 feePct = timeDiff * licenseFee / ONE_YEAR;
=> therefore we have: feePct = licenseFee which is 1e15 (0.1%) according to our assumptions

uint256 fee = startSupply * feePct / (BASE - feePct);
insert the values => fee = 1e18 * 1e15 / (1e18 - 1e15) => (factor out 1e15) => fee = 1e15 * 1e18 / (1e15 * ( 1e3 - 1) => (cancel 1e15) => 1e18 / ( 1e3 - 1)

math: if we increase the divisor but the dividend stays the same we get a smaller number e.g. (1 / (2-1)) is bigger than (1 / 2)
apply this here => 1e18 / ( 1e3 - 1) > 1e18 / 1e3 => 1e18 / ( 1e3 - 1) > 1e15  this shows that the fee is higher than 1e15


https://github.com/code-423n4/2021-12-defiprotocol/blob/205d3766044171e325df6a8bf2e79b37856eece1/contracts/contracts/Basket.sol#L133
https://github.com/code-423n4/2021-12-defiprotocol/blob/205d3766044171e325df6a8bf2e79b37856eece1/contracts/contracts/Basket.sol#L141

## Tools Used
Manual Analysis

## Recommended Mitigation Steps
- only divide by BASE

