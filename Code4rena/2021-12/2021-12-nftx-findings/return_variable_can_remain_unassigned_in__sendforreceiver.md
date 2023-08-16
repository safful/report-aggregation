## Tags

- bug
- 2 (Med Risk)
- resolved
- sponsor confirmed

# [Return variable can remain unassigned in _sendForReceiver](https://github.com/code-423n4/2021-12-nftx-findings/issues/121) 

# Handle

sirhashalot


# Vulnerability details

## Impact

The `_sendForReceiver()` function only sets a return function in the "if" code block, not the "else" case. If the "else" case is true, no value is returned. The result of this oversight is that the `_sendForReceiver()` function called from the `distribute()` function could sucessfully enter its `else` block if a receiver has `isContract` set to False and successfully transfer the `amountToSend` value. The `ditribute()` function will then have `leftover > 0` and send `currentTokenBalance` to the treasury. This issue is partially due to [Solidity using implicit returns](https://github.com/ethereum/solidity/issues/2951), so if no bool value is explicitly returned, the default bool value of False will be returned.

This problem currently occurs for any receiver with `isContract` set to False. The `_addReceiver` function allows for `isContract` to be set to False, so such a condition should not result in tokens being sent to the treasury as though it was an emergency scenario.

## Proof of Concept

The `else` block is missing a return value
https://github.com/code-423n4/2021-12-nftx/blob/194073f750b7e2c9a886ece34b6382b4f1355f36/nftx-protocol-v2/contracts/solidity/NFTXSimpleFeeDistributor.sol#L167-L169

## Tools Used

VS Code "Solidity Visual Developer" extension

## Recommended Mitigation Steps

Verify that functions with a return value do actually return a value in all cases. Adding the line `return true;` can be added to the end of the `else` block as one way to resolve this.

Alternatively, if `isContract` should never be set to False, the code should be designed to prevent a receiver from being added with this value.

