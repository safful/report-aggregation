## Tags

- bug
- 2 (Med Risk)
- disagree with severity
- sponsor confirmed
- Notional

# [fCash of the wrong maturity and asset can be sent to wrapper address before wrapper is deployed ](https://github.com/code-423n4/2022-06-notional-coop-findings/issues/115) 

# Lines of code

https://github.com/code-423n4/2022-06-notional-coop/blob/6f8c325f604e2576e2fe257b6b57892ca181509a/notional-wrapped-fcash/contracts/wfCashLogic.sol#L131


# Vulnerability details

## Impact
Minting becomes impossible 

## Proof of Concept
onERC1155Received is only called when the size of the code deployed at the address contains code. Since create2 is used to deploy the contract, the address can be calculated before the contract is deployed. A malicious actor could send the address fCash of a different maturity or asset before the contract is deployed and since nothing has been deployed, onERC1155Received will not be called and the address will accept the fCash. After the contract is deployed and correct fCash is sent to the address, onERC1155Received will check the length of the assets held by the address and it will be more than 1 (fCash of correct asset and maturity and fCash with wrong maturity or asset sent before deployment). This will cause the contract to always revert essentially breaking the mint completely. 

## Tools Used

## Recommended Mitigation Steps
When the contract is created create a function that reads how many fCash assets are at the address and send them away if they aren't of the correct asset and maturity

