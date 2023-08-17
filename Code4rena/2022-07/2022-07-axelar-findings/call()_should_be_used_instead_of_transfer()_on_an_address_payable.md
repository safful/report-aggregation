## Tags

- bug
- QA (Quality Assurance)
- sponsor confirmed

# [CALL() SHOULD BE USED INSTEAD OF TRANSFER() ON AN ADDRESS PAYABLE](https://github.com/code-423n4/2022-07-axelar-findings/issues/4) 

# Lines of code

https://github.com/code-423n4/2022-07-axelar/blob/main/contracts/gas-service/AxelarGasService.sol#L128
https://github.com/code-423n4/2022-07-axelar/blob/main/contracts/gas-service/AxelarGasService.sol#L144
https://github.com/code-423n4/2022-07-axelar/blob/main/contracts/gas-service/AxelarGasService.sol#L158


# Vulnerability details


# Vulnerability details


## Impact
The use of the deprecated transfer() function for an address will inevitably make the transaction fail when:

The claimer smart contract does not implement a payable function.
The claimer smart contract does implement a payable fallback which uses more than 2300 gas unit.
The claimer smart contract implements a payable fallback function that needs less than 2300 gas units but is called through proxy, raising the call’s gas usage above 2300.
Additionally, using higher than 2300 gas might be mandatory for some multisig wallets.
Whenever the user either fails to implement the payable fallback function or cumulative gas cost of the function sequence invoked on a native token transfer exceeds 2300 gas consumption limit the native tokens sent end up undelivered and the corresponding user funds return functionality will fail each time.
The impact would mean that any contracts receiving funds would potentially be unable to retrieve funds from the swap.

## Recommended Mitigation Steps
use call() to send eth , re-entrancy has been accounted for in all functions that reference Solidity's transfer() . This has been done by using a re-entrancy guard, therefore, we can rely on msg.sender.call.value(amount)` or using the OpenZeppelin Address.sendValue library


Relevant links:
https://github.com/code-423n4/2021-04-meebits-findings/issues/2
https://twitter.com/hacxyk/status/1520715516490379264?s=21&t=fnhDkcC3KpE_kJE8eLiE2A
https://consensys.net/diligence/blog/2019/09/stop-using-soliditys-transfer-now/