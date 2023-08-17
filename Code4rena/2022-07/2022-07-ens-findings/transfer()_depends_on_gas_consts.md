## Tags

- bug
- 2 (Med Risk)
- disagree with severity
- sponsor confirmed

# [transfer() depends on gas consts](https://github.com/code-423n4/2022-07-ens-findings/issues/133) 

# Lines of code

https://github.com/code-423n4/2022-07-ens/blob/ff6e59b9415d0ead7daf31c2ed06e86d9061ae22/contracts/ethregistrar/ETHRegistrarController.sol#L183-L185
https://github.com/code-423n4/2022-07-ens/blob/ff6e59b9415d0ead7daf31c2ed06e86d9061ae22/contracts/ethregistrar/ETHRegistrarController.sol#L204


# Vulnerability details

## Impact
`transfer()` forwards 2300 gas only, which may not be enough in future if the recipient is a contract and gas costs change. it could break existing contracts functionality.

## Proof of Concept
`.transfer` or `.send` method, only 2300 gas will be “forwarded” to fallback function. Specifically, the SLOAD instruction, will go from costing 200 gas to 800 gas.

if any smart contract has a functionality of register ens and it has fallback function which is making some state change in contract on ether receive, it could use more than 2300 gas and revert every transaction

for reference checkout this,
https://docs.soliditylang.org/en/v0.8.15/security-considerations.html?highlight=transfer#sending-and-receiving-ether
https://consensys.net/diligence/blog/2019/09/stop-using-soliditys-transfer-now/

## Tools Used
Manual Analysis

## Recommended Mitigation Steps

use `.call` insted `.transfer`

     (bool success, ) = msg.sender.call.value(amount)("");
     require(success, "Transfer failed.");


