## Tags

- bug
- 1 (Low Risk)
- resolved
- sponsor confirmed

# [Anyone can crash transferTo](https://github.com/code-423n4/2022-01-openleverage-findings/issues/261) 

# Handle

pauliax


# Vulnerability details

## Impact
function transferTo allows transferring amount from beneficiary to any address. However, 'to' is considered valid when it does not have an amount locked yet:
```solidity
 function transferTo(address to, uint amount) external
 ...
   require(releaseVars[to].amount == 0, 'to is exist');
 ```
It locks this amount for releaseVars[beneficiary].endTime. Because the blockchain is public, a malicious actor could monitor the mempool, and crash any attempt of transferTo by frontrunning it and calling transferTo with the smallest fraction (dust) from his own address to the 'to' address, making it unavailable to receive new locks for some time (even 4 years is possible?).

## Recommended Mitigation Steps
A few possible solutions would be to introduce a reasonable minimum amount to transfer or add a 2-step approval, where 'to' first have to approve the beneficiary.

