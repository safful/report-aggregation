## Tags

- bug
- 2 (Med Risk)
- resolved
- sponsor confirmed

# [All AMMs have to be past nextFundingTime to update](https://github.com/code-423n4/2022-02-hubble-findings/issues/130) 

# Lines of code

https://github.com/code-423n4/2022-02-hubble/blob/ed1d885d5dbc2eae24e43c3ecbf291a0f5a52765/contracts/AMM.sol#L348


# Vulnerability details

# Impact

settleFunding calls will revert until all AMMs are ready to be updated.

# Proof of Concept

1. AMM 1 has a nextFundingTime of now. AMM 2 has a nextFundingTime in 30 minutes. AMM 1 won't be able to be updated until after AMM 2's nextFundingTime elapses.

# Mitigation
You shouldn't revert at the place mentioned in the links to affected code. Just return so that the other AMMs can still get updated.

