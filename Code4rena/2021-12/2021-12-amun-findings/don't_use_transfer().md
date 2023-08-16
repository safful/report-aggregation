## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [Don't use transfer()](https://github.com/code-423n4/2021-12-amun-findings/issues/175) 

# Handle

GiveMeTestEther


# Vulnerability details

## Impact
Use of transfer() might render ETH impossible to withdraw becuase after istanbul hardfork , there is increases in the gas cost of the SLOAD operation and therefore breaks some existing smart contracts.Those contracts will break because their fallback functions used to consume less than 2300 gas, and they’ll now consume more, since 2300 the amount of gas a contract’s fallback function receives if it’s called via Solidity’s transfer() or send() methods.
Any smart contract that uses transfer() or send() is taking a hard dependency on gas costs by forwarding a fixed amount of gas: 2300.

https://consensys.net/diligence/blog/2019/09/stop-using-soliditys-transfer-now/

## Proof of Concept

2021-12-amun\contracts\basket\contracts\singleJoinExit\EthSingleTokenJoin.sol: msg.sender.transfer(remainingIntermediateBalance);

2021-12-amun\contracts\basket\contracts\singleJoinExit\EthSingleTokenJoinV2.sol: msg.sender.transfer(remainingIntermediateBalance);

2021-12-amun\contracts\basket\contracts\singleJoinExit\SingleNativeTokenExit.sol: msg.sender.transfer(intermediateTokenBalance);

2021-12-amun\contracts\basket\contracts\singleJoinExit\SingleNativeTokenExitV2.sol: msg.sender.transfer(intermediateTokenBalance);.

## Tools Used
Manual Analysis

## Recommended Mitigation Steps
-use call() to send ETH

