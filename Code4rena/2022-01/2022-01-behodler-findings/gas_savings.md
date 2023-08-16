## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Gas savings](https://github.com/code-423n4/2022-01-behodler-findings/issues/217) 

# Handle

csanuragjain


# Vulnerability details

## Impact
Gas savings

## Proof of Concept

1. Navigate to contract at https://github.com/code-423n4/2022-01-behodler/blob/main/contracts/DAO/LimboDAO.sol

2. Observe that in burnAsset function, fateCreated can be initialized with 0 instead of fateState[_msgSender()].fateBalance which takes more gas. Actual value of fateCreated is decided by if-else condition

3. Observe that in vote function isLive modifier should be before incrementFate as if contract is not live then there is no meaning of incrementFate

4. Observe that in vote function below nested condition can be placed beforehand as if this happens further execution is not required

```
if (block.timestamp - currentProposalState.start > proposalConfig.votingDuration)
```

5. Observe that previousProposalState is never used and can be removed

