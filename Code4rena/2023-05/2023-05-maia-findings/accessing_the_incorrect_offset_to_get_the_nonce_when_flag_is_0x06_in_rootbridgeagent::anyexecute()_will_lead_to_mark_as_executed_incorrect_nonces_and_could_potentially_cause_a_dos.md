## Tags

- bug
- 3 (High Risk)
- primary issue
- satisfactory
- selected for report
- sponsor confirmed
- upgraded by judge
- edited-by-warden
- H-26

# [Accessing the incorrect offset to get the nonce when flag is 0x06 in RootBridgeAgent::anyExecute() will lead to mark as executed incorrect nonces and could potentially cause a DoS](https://github.com/code-423n4/2023-05-maia-findings/issues/267) 

# Lines of code

https://github.com/code-423n4/2023-05-maia/blob/main/src/ulysses-omnichain/RootBridgeAgent.sol#L1083-L1116


# Vulnerability details

## Impact
- Not reading the correct offset where the `nonce` is located can lead to set as executed the incorrect nonce, which will cause unexpected behavior and potentially a DoS when attempting to execute a `nonce` that incorrectly was marked as already executed.

## Proof of Concept
- The structure of the [data is encoded as detailed in the `IRootBridgeAgent` contract](https://github.com/code-423n4/2023-05-maia/blob/main/src/ulysses-omnichain/interfaces/IRootBridgeAgent.sol#L144)
```solidity
 *          |            Flag               |        Deposit Info        |             Token Info             |   DATA   |  Gas Info   |
 *          |           1 byte              |         4-25 bytes         |     3 + (105 or 128) * n bytes     |   ---	 |  32 bytes   |
 *          |_______________________________|____________________________|____________________________________|__________|_____________|

 *          | callOutSignedMultiple = 0x6   |   20b + 1b(n) + 4b(nonce)  |      32b + 32b + 32b + 32b + 3b 	  |   ---	 |  16b + 16b  |
```

- The actual encoding of the data happens on the `BranchBridgeAgent` contract, [on these lines](https://github.com/code-423n4/2023-05-maia/blob/main/src/ulysses-omnichain/BranchBridgeAgent.sol#L288-L301)

- Based on the data structure we can decode and determine on what offset is located what data
  - `data[0]`       => flag
  - `data[1:21]`    => an address
  - `data[21]`      => hTokens.length
  - `data[22:26]`   => The 4 bytes of the nonce

- So, when flag is `0x06`, the nonce is located at the offset `data[22:26]`, that indicates that the current offset that is been accessed is wrong [(`data[PARAMS_START_SIGNED:25]` === `data[21:]`)](https://github.com/code-423n4/2023-05-maia/blob/main/src/ulysses-omnichain/RootBridgeAgent.sol#L1083-L1085)

## Tools Used
Manual Audit

## Recommended Mitigation Steps
- Make sure to read the `nonce` from the correct offset, based on the [data structure as explain in the `IRootBridgeAgent` contract]()

- For `flag 0x06`, read the offset as follows, either of the two options are correct:
  - **`nonce` is located at: `data[22:26]`**
```solidity
nonce = uint32(bytes4(data[PARAMS_START_SIGNED + PARAMS_START : 26]));
nonce = uint32(bytes4(data[22:26]));
```





## Assessed type

en/de-code