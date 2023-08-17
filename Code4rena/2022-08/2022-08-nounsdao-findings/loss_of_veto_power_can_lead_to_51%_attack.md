## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Loss of Veto Power can Lead to 51% Attack](https://github.com/code-423n4/2022-08-nounsdao-findings/issues/315) 

# Lines of code

https://github.com/code-423n4/2022-08-nounsdao/blob/45411325ec14c6d747b999a40367d3c5109b5a89/contracts/governance/NounsDAOLogicV2.sol#L156
https://github.com/code-423n4/2022-08-nounsdao/blob/45411325ec14c6d747b999a40367d3c5109b5a89/contracts/governance/NounsDAOLogicV1.sol#L150
https://github.com/code-423n4/2022-08-nounsdao/blob/45411325ec14c6d747b999a40367d3c5109b5a89/contracts/governance/NounsDAOLogicV2.sol#L839-L845
https://github.com/code-423n4/2022-08-nounsdao/blob/45411325ec14c6d747b999a40367d3c5109b5a89/contracts/governance/NounsDAOLogicV1.sol#L637-L643


# Vulnerability details

## Impact
The veto power is import functionality for current NounsDAO in order to protect their treasury from malicious proposals. 
However there is lack of zero address check and lack of 2 step address changing process for vetoer address.
This might lead to Nounders losing their veto power unintentionally and open to 51% attack which can drain their entire treasury.

Refrence from Nouns DAO contest documents:
https://dialectic.ch/editorial/nouns-governance-attack
https://dialectic.ch/editorial/nouns-governance-attack-2

## Proof of Concept
Lack of 0-address check for vetoer address at initialize() and _setVetoer() of NounsDAOLogicV2.sol and NounsDAOLogicV1.sol.
Also it is better to make changing address process of vetoer at _setVetoer() into 2-step process to avoid accidently setting
vetoer to zero address or any other arbitrary addresses and end up burning/losing veto power unintentionally.

1. vetoer address of initialize() of NounsDAOLogicV2.sol, NounsDAOLogicV1.sol
https://github.com/code-423n4/2022-08-nounsdao/blob/45411325ec14c6d747b999a40367d3c5109b5a89/contracts/governance/NounsDAOLogicV2.sol#L156
https://github.com/code-423n4/2022-08-nounsdao/blob/45411325ec14c6d747b999a40367d3c5109b5a89/contracts/governance/NounsDAOLogicV1.sol#L150

2. vetoer address of _setVetoer() of NounsDAOLogicV2.sol, NounsDAOLogicV1.sol
https://github.com/code-423n4/2022-08-nounsdao/blob/45411325ec14c6d747b999a40367d3c5109b5a89/contracts/governance/NounsDAOLogicV2.sol#L839-L845
https://github.com/code-423n4/2022-08-nounsdao/blob/45411325ec14c6d747b999a40367d3c5109b5a89/contracts/governance/NounsDAOLogicV1.sol#L637-L643

## Tools Used
Manual Analysis

## Recommended Mitigation Steps
Add zero address check for vetoer address at initialize().
Also change _setVetoer() vetoer address changing process to 2-step process like explained below.

First make the _setVetoer() function approve a new vetoer address as a pending vetoer.
Next that pending vetoer has to claim the ownership in a separate transaction to be a new vetoer.