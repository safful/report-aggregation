## Tags

- bug
- 2 (Med Risk)
- disagree with severity
- sponsor confirmed
- valid

# [Incorrect initialization of smart contracts with Access Control issue](https://github.com/code-423n4/2022-08-rigor-findings/issues/6) 

# Lines of code

https://github.com/code-423n4/2022-08-rigor/blob/5ab7ea84a1516cb726421ef690af5bc41029f88f/contracts/HomeFiProxy.sol#L216-L230
https://github.com/code-423n4/2022-08-rigor/blob/5ab7ea84a1516cb726421ef690af5bc41029f88f/contracts/Community.sol#L102-L119
https://github.com/code-423n4/2022-08-rigor/blob/5ab7ea84a1516cb726421ef690af5bc41029f88f/contracts/DebtToken.sol#L43-L58
https://github.com/code-423n4/2022-08-rigor/blob/5ab7ea84a1516cb726421ef690af5bc41029f88f/contracts/Disputes.sol#L74-L81
https://github.com/code-423n4/2022-08-rigor/blob/5ab7ea84a1516cb726421ef690af5bc41029f88f/contracts/HomeFi.sol#L92-L120
https://github.com/code-423n4/2022-08-rigor/blob/5ab7ea84a1516cb726421ef690af5bc41029f88f/contracts/Project.sol#L94-L105
https://github.com/code-423n4/2022-08-rigor/blob/5ab7ea84a1516cb726421ef690af5bc41029f88f/contracts/ProjectFactory.sol#L45-L55


# Vulnerability details

## Impact
All next Impact depends on actions and attention from developers when deployed
- Loss of funds 
- Failure of the protocol, with the need for redeploy
- Loss of control over protocol elements (some smart contracts)
- The possibility of replacing contracts and settings with harmful ones
And other things that come out of it...

## Proof of Concept

For a proper understanding of Proof of Concept, you need to understand the following things:
1) Hardhat does not stop the process with a deploy and does not show failed transactions if they have occurred in some cases 
2) Malicious agents can trace the protocol deployment transactions and insert their own transaction between them

Reason:
- [During deploy TransparentUpgradeableProxy's](https://github.com/code-423n4/2022-08-rigor/blob/5ab7ea84a1516cb726421ef690af5bc41029f88f/contracts/HomeFiProxy.sol#L216-L230) initialize method for initializing contracts not called. The third parameter responsible for this is an empty string. This causes the initialization process itself to be **delayed**

- Contract initialization methods have no check over who calls them
Example [ProjectFactory.sol#L45-L55](https://github.com/code-423n4/2022-08-rigor/blob/5ab7ea84a1516cb726421ef690af5bc41029f88f/contracts/ProjectFactory.sol#L45-L55)

**Also suitable for other contracts, strings are attached in Links to affected code **

Example of exploiting the vulnerability **Failure of the protocol, with the need for redeploy** && **Loss of control over protocol elements (some smart contracts)** :
1) User listen transaction in mempool, etherscan, transaction in block etc 
2) Finds the moment of deployment and sends the transaction for setup his HomeFi address in Disputes contract:
  Just he call initialize method and put his _homeFi parameter
3) In the event that hardhat tracked a failed transaction, the deployment will stop and you will need to start over. If the hardhead misses it and the developers do not check the result and the setting, access to this part will be lost and fix is needed

Example of exploiting the vulnerability **Loss of funds**:
1) User listen transaction in mempool, etherscan, transaction in block for listne when HomeFi will deployed
2) Send transaction for initialize [HomeFi](https://github.com/code-423n4/2022-08-rigor/blob/5ab7ea84a1516cb726421ef690af5bc41029f88f/contracts/HomeFi.sol#L92-L120) with his _treasury address
3) Transfer the admin ownership the right to the real address to divert the eyes
4) The address of the treasury remains with the attacker
5) The protocol fees (fee) will be [transfered](https://github.com/code-423n4/2022-08-rigor/blob/5ab7ea84a1516cb726421ef690af5bc41029f88f/contracts/Community.sol#L443) to the attacker's address until it is detected


## Recommended Mitigation Steps

Carry out checks at the initialization stage or redesign the deployment process with the initialization of contracts during deployment

