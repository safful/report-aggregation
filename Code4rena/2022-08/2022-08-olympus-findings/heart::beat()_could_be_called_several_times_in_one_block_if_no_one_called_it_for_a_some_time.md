## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Heart::beat() could be called several times in one block if no one called it for a some time](https://github.com/code-423n4/2022-08-olympus-findings/issues/79) 

# Lines of code

https://github.com/code-423n4/2022-08-olympus/blob/main/src/policies/Heart.sol#L92
https://github.com/code-423n4/2022-08-olympus/blob/main/src/policies/Heart.sol#L103


# Vulnerability details

## Impact
`beat()` function is allowed to be called by anyone once in `frequency()` period. The purpose of it is to update the prices and do another operations related to bond market. User who ran it are rewarded. There is no need to run this function more then 1 time in `frequency()` period.
However if `beat()` was last time called more then `frequency()` time ago then user can execute `beat()` function `(block.timestamp - lastBeat)/frequency()` times in a row in same block and get rewards.

## Proof of Concept
https://github.com/code-423n4/2022-08-olympus/blob/main/src/policies/Heart.sol#L92
https://github.com/code-423n4/2022-08-olympus/blob/main/src/policies/Heart.sol#L103


## Recommended Mitigation Steps
https://github.com/code-423n4/2022-08-olympus/blob/main/src/policies/Heart.sol#L103
Change this line to `lastBeat = block.timestamp - (block.timestamp - lastBeat) % frequency();`
So no matter how much time the `beat()` was no called, it is possible to call it only once per `frequency()`.
