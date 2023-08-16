## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed
- resolved

# [Shadowing Local Variables found in RCOrderbook.sol](https://github.com/code-423n4/2021-06-realitycards-findings/issues/124) 

# Handle

maplesyrup


# Vulnerability details

## Impact
1 - Low Risk
   - Possible incorrect use of variables are at stake which may have bad side effects to the contract if implemented incorrectly.

## Proof of Concept

According to the Slither-analyzer documentation (https://github.com/crytic/slither/wiki/Detector-Documentation#local-variable-shadowing), shadowing local variables is naming conventions found in two or more variables that are similar. Although they do not pose any immediate risk to the contract, incorrect usage of the variables is possible and can cause serious issues if the developer does not pay close attention. 

It is recommended that the naming of the following variables should be changed slightly to avoid any confusion:

 -------------------------------------------------------------------

RCOrderbook._updateBidInOrderbook(address,address,uint256,uint256,uint256,RCOrderbook.Bid)._owner 

(contracts/RCOrderbook.sol line(s)#358) shadows:

Ownable._owner <------(state variable)

(node_modules/@openzeppelin/contracts/access/Ownable.sol line(s)#19) 

 -------------------------------------------------------------------

RCOrderbook.closeMarket()._owner 

(contracts/RCOrderbook.sol line(s)#639) shadows:

Ownable._owner <------(state variable)

(node_modules/@openzeppelin/contracts/access/Ownable.sol line(s)#19)

 -------------------------------------------------------------------

## Tools Used

Solidity Compiler 0.8.4
Hardhat v2.3.3
Slither v0.8.0

Compiled, Tested, Deployed contracts on a local hardhat network.

Ran Slither-analyzer for further detecting and testing.

## Recommended Mitigation Steps

(Worked best under python venv)
1. Clone Project Repository
2. Run Project against Hardhat network;
   compile and run default test on contracts.
3. Installed slither analyzer:
  https://github.com/crytic/slither
4. Ran [$ slither .] against RCOrderbook.sol and all contracts to verify results

