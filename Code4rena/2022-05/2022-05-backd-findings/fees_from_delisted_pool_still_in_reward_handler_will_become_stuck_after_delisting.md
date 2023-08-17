## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Fees from delisted pool still in reward handler will become stuck after delisting](https://github.com/code-423n4/2022-05-backd-findings/issues/135) 

# Lines of code

https://github.com/code-423n4/2022-05-backd/blob/2a5664d35cde5b036074edef3c1369b984d10010/protocol/contracts/Controller.sol#L62-L76


# Vulnerability details

## Impact
Unclaimed fees from pool will be stuck

## Proof of Concept
When delisting a pool the pool's reference is removed from address provider:

https://github.com/code-423n4/2022-05-backd/blob/2a5664d35cde5b036074edef3c1369b984d10010/protocol/contracts/Controller.sol#L63

Burning fees calls a dynamic list of all pools which no longer contains the delisted pool:

https://github.com/code-423n4/2022-05-backd/blob/2a5664d35cde5b036074edef3c1369b984d10010/protocol/contracts/RewardHandler.sol#L39

Since the list no longer contains the pool those fees will not be processed and will remain stuck in the contract 

## Tools Used

## Recommended Mitigation Steps
Call burnFees() before delisting a pool

