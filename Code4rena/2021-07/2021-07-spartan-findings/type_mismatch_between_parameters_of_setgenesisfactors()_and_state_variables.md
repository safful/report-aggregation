## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed
- disagree with severity

# [Type mismatch between parameters of setGenesisFactors() and state variables](https://github.com/code-423n4/2021-07-spartan-findings/issues/95) 

# Handle

0xRajeev


# Vulnerability details

## Impact

The state variables corresponding to setGenesisFactors() parameters _coolOff, _daysToEarn, _majorityFactor, _daoClaim and_daoFee are declared to be uint256 but are set using these parameters that are uint32. While it’s unlikely that these will need values > uint32, this leads to wastage of storage slots and gas. 

## Proof of Concept

https://github.com/code-423n4/2021-07-spartan/blob/e2555aab44d9760fdd640df9095b7235b70f035e/contracts/Dao.sol#L128

https://github.com/code-423n4/2021-07-spartan/blob/e2555aab44d9760fdd640df9095b7235b70f035e/contracts/Dao.sol#L21-L24

https://github.com/code-423n4/2021-07-spartan/blob/e2555aab44d9760fdd640df9095b7235b70f035e/contracts/Dao.sol#L19


## Tools Used

Manual Analysis

## Recommended Mitigation Steps

The state variables can be declared uint32 to fit all five of them in a single slot and this will lead to efficient SSTOREs because they are set together. If values > uint32 are relevant, then the parameter types of setter setGenesisFactors() have to be changed.

