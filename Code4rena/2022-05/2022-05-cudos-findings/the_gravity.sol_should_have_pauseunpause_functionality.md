## Tags

- bug
- 2 (Med Risk)
- resolved
- sponsor confirmed

# [The Gravity.sol should have pause/unpause functionality](https://github.com/code-423n4/2022-05-cudos-findings/issues/139) 

# Lines of code

https://github.com/code-423n4/2022-05-cudos/blob/main/solidity/contracts/Gravity.sol#L175


# Vulnerability details

## Impact

In case a hack is occuring or an exploit is discovered, the team (or validators in this case) should be able to pause
functionality until the necessary changes are made to the system. Additionally, the gravity.sol contract should be manged by proxy so that upgrades can be made by the validators.

Because an attack would probably span a number of blocks, a method for pausing the contract would be able to interrupt any such attack if discovered.

To use a thorchain example again, the team behind thorchain noticed an attack was going to occur well before
the system transferred funds to the hacker. However, they were not able to shut the system down fast enough.
(According to the incidence report here: https://github.com/HalbornSecurity/PublicReports/blob/master/Incident%20Reports/Thorchain_Incident_Analysis_July_23_2021.pdf)


## Proof of Concept

https://github.com/code-423n4/2022-05-cudos/blob/main/solidity/contracts/Gravity.sol#L175

## Tools Used

Code Review

## Recommended Mitigation Steps

Pause functionality on the contract would have helped secure the funds quickly.


