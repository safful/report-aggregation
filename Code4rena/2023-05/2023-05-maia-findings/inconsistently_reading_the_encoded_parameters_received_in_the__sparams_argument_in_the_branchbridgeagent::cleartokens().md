## Tags

- bug
- 2 (Med Risk)
- disagree with severity
- primary issue
- satisfactory
- selected for report
- sponsor confirmed
- M-41

# [Inconsistently reading the encoded parameters received in the _sParams argument in the BranchBridgeAgent::clearTokens()](https://github.com/code-423n4/2023-05-maia-findings/issues/173) 

# Lines of code

https://github.com/code-423n4/2023-05-maia/blob/main/src/ulysses-omnichain/BranchBridgeAgent.sol#L565-L634


# Vulnerability details

## Impact
- token addresses could be computed wrong which could lead to get stuck the tokens in the root chain

## Proof of Concept
- The function [clearTokens()](https://github.com/code-423n4/2023-05-maia/blob/main/src/ulysses-omnichain/BranchBridgeAgent.sol#L565-L634) is called from the [BranchBridgeAgentExecutor::executeWithSettlementMultiple() function](https://github.com/code-423n4/2023-05-maia/blob/main/src/ulysses-omnichain/BranchBridgeAgentExecutor.sol#L116-L157), which is used when the settlement flag is 2 "Multiple Settlements"

- As per the [documentation about the messaging layer written in the `IBranchBridgeAgent` contract](https://github.com/code-423n4/2023-05-maia/blob/main/src/ulysses-omnichain/interfaces/IBranchBridgeAgent.sol#L112-L136), [when the flag is 2, the structure of the token info is as follows](https://github.com/code-423n4/2023-05-maia/blob/main/src/ulysses-omnichain/interfaces/IBranchBridgeAgent.sol#L135):

```
 *           - ht = hToken
 *           - t = Token
 *           - A = Amount
 *           - D = Destination
 *           - b = bytes
 *           - n = number of assets
 *           ________________________________________________________________________________________________________________________________
 *          |            Flag               |           Deposit Info           |             Token Info             |   DATA   |  Gas Info   |
 *          |           1 byte              |            4-25 bytes            |        (105 or 128) * n bytes      |   ---	   |  16 bytes   |
 *          |                               |                                  |            hT - t - A - D          |          |             |
 *          |_______________________________|__________________________________|____________________________________|__________|_____________|

 *          | callOutMulti = 0x2            |  1b(n) + 20b(recipient) + 4b     |   	     32b + 32b + 32b + 32b      |   ---	   |     16b     |
```

- 3 of the 4 parameters encoded in _sParams ([hTokens](https://github.com/code-423n4/2023-05-maia/blob/main/src/ulysses-omnichain/BranchBridgeAgent.sol#L582-L593), [amounts](https://github.com/code-423n4/2023-05-maia/blob/main/src/ulysses-omnichain/BranchBridgeAgent.sol#L604-L612) and [deposits](https://github.com/code-423n4/2023-05-maia/blob/main/src/ulysses-omnichain/BranchBridgeAgent.sol#L613-L621)) reads the whole 32 bytes and [tokens read only 20 bytes](https://github.com/code-423n4/2023-05-maia/blob/main/src/ulysses-omnichain/BranchBridgeAgent.sol#L594-L603).


## Tools Used
Manual Audit

## Recommended Mitigation Steps

- Standardize the way to read parameters from the received _sParams, if all parameters are bytes32, make sure to read all the bytes corresponding to such a parameter, and from there do the required conversions to another data types.
```solidity
function clearTokens(bytes calldata _sParams, address _recipient)
    ...
{
    ...
        _tokens[i] = address(
            uint160(
                bytes20(
                    bytes32(
                        _sParams[
                            PARAMS_TKN_START + PARAMS_ENTRY_SIZE * uint16(i + numOfAssets) + 12:
                                PARAMS_TKN_START + PARAMS_ENTRY_SIZE * uint16(PARAMS_START + i + numOfAssets)
                        ]
                    )     
                )
            )
        );
    ...
}
```


## Assessed type

en/de-code