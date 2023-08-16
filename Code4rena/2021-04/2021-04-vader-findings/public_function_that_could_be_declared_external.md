## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed
- addressed

# [Public function that could be declared external](https://github.com/code-423n4/2021-04-vader-findings/issues/14) 

# Handle

JMukesh


# Vulnerability details

## Impact
public functions that are never called by the contract should be declared external to save gas.

## Proof of Concept
 1. In Vault.sol  -- > init() and grant()
               
    https://github.com/code-423n4/2021-04-vader/blob/main/vader-protocol/contracts/Vault.sol#L45

https://github.com/code-423n4/2021-04-vader/blob/main/vader-protocol/contracts/Vault.sol#L68

2. Vader.sol -- > burn() 

 https://github.com/code-423n4/2021-04-vader/blob/main/vader-protocol/contracts/Vader.sol#L146

3. Utils.sol -- > init(),  getProtection()

 https://github.com/code-423n4/2021-04-vader/blob/main/vader-protocol/contracts/Utils.sol#L30

4. Router.sol -- >

init(address,address,address) 
getVADERAmount(uint256) 
getUSDVAmount(uint256) 
borrow(uint256,address,address) 
repay(uint256,address,address) 
checkLiquidate()
getSystemCollateral(address,address) 
getSystemDebt(address,address)
getSystemInterestPaid() 

https://github.com/code-423n4/2021-04-vader/blob/main/vader-protocol/contracts/Router.sol#L77

5. Pools.sol

init(address,address,address,address) 
isMember(address) 
isSynth(address) 

https://github.com/code-423n4/2021-04-vader/blob/main/vader-protocol/contracts/Pools.sol

6. Dao.sol

init(address,address,address) 
newGrantProposal(address,uint256) 
newAddressProposal(address,string) 
voteProposal(uint256) 
cancelProposal(uint256,uint256) 
finaliseProposal(uint256)

https://github.com/code-423n4/2021-04-vader/blob/main/vader-protocol/contracts/DAO.sol#L46
	

## Tools Used

slither

## Recommended Mitigation Steps

use external instead of public visibility to save gas

