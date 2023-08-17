## Tags

- bug
- 2 (Med Risk)
- resolved
- sponsor confirmed

# [Order duration can be set to 0 by Malicious maker](https://github.com/code-423n4/2022-06-putty-findings/issues/107) 

# Lines of code

https://github.com/code-423n4/2022-06-putty/blob/main/contracts/src/PuttyV2.sol#L287


# Vulnerability details

## Impact
A malicious maker can set a minimum order duration as 0 which means order will instantly expire after filling. Taker will get only the withdraw option and that too with fees on strike price, thus forcing the taker to lose money in this meaningless transaction

## Proof of Concept
!. Maker creates an order with zero Order duration
2. Taker fills this order but the order instantly expires since duration was 0
3. Taker gets the only option to withdraw with fees on strike price

## Recommended Mitigation Steps
Enforce atleast x days of duration

