## Tags

- bug
- 0 (Non-critical)
- resolved
- sponsor confirmed

# [Limbo, LimboDAO and FlashGovernanceArbiter events aren't indexed](https://github.com/code-423n4/2022-01-behodler-findings/issues/249) 

# Handle

hyh


# Vulnerability details

## Impact

No events in Limbo, LimboDAO and FlashGovernanceArbiter contracts are indexed, so their filtering is disabled, which makes it harder to programmatically use the system

## Proof of Concept

Contract's events don't have indices:

Limbo:

https://github.com/code-423n4/2022-01-behodler/blob/main/contracts/Limbo.sol#L253-260

FlashGovernanceArbiter:

https://github.com/code-423n4/2022-01-behodler/blob/main/contracts/DAO/FlashGovernanceArbiter.sol#L22

LimboDAO:

https://github.com/code-423n4/2022-01-behodler/blob/main/contracts/DAO/LimboDAO.sol#L56-61

## Recommended Mitigation Steps

Consider adding the indices to the key parameters, first of all to the addresses of the tokens and accounts broadcasted


