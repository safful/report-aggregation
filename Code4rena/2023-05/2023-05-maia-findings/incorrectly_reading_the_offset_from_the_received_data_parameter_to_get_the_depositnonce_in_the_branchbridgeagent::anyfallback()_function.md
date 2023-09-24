## Tags

- bug
- 3 (High Risk)
- primary issue
- satisfactory
- selected for report
- sponsor confirmed
- edited-by-warden
- H-32

# [Incorrectly reading the offset from the received data parameter to get the depositNonce in the BranchBridgeAgent::anyFallback() function](https://github.com/code-423n4/2023-05-maia-findings/issues/169) 

# Lines of code

https://github.com/code-423n4/2023-05-maia/blob/main/src/ulysses-omnichain/BranchBridgeAgent.sol#L1272-L1304


# Vulnerability details

## Impact
- Not reading the correct offset where the `depositNonce` is located can lead to set the status of the wrong deposit to Failed when the [_clearDeposit() function](https://github.com/code-423n4/2023-05-maia/blob/main/src/ulysses-omnichain/BranchBridgeAgent.sol#L988-L996) is called.
  - The consequences of setting the incorrect `depositNonce` to False can be:
    - Getting stuck the deposits of the real `depositNonce` that is sent to the anyFallback() because that `depositNonce` won't be marked as Failed.
    - Causing troubles to other `depositNonces` that should not be marked as Failed.

## Proof of Concept
- The structure of the data was encoded depending on the type of operation, that means that **the `depositNonce` will be located at a different offset depending the flag.**
- To see where exactly the `depositNonce` is located, is required to check the corresponding operation where the data was packed
  - Depending on the type of operation (flag) it will be the function we'll need to analyze to determine the correct offset where the depositNonce was packed.
- Let's analyze flag by flag the encoded data and determine the correct offset of the `depositNonce` for each flag

1. **`flag == 0x00`**
    - When [encoding the data for the flag 0x00](https://github.com/code-423n4/2023-05-maia/blob/main/src/ulysses-omnichain/BranchBridgeAgent.sol#L480-L481), we can see that **the `depositNonce` is located at the data[1:5]**
```solidity
bytes memory packedData =
          abi.encodePacked(bytes1(0x00), depositNonce, _params, gasToBridgeOut, _remoteExecutionGas);

// data[0]    ==> flag === 0x00
// data[1:5]  ==> depositNonce
```

2. **`flag == 0x01`**
    - When [encoding the data for the flag 0x01](https://github.com/code-423n4/2023-05-maia/blob/main/src/ulysses-omnichain/BranchBridgeAgent.sol#L656-L657), we can see that **the `depositNonce` is located at the data[1:5]**
```solidity
bytes memory packedData =
        abi.encodePacked(bytes1(0x01), depositNonce, _params, _gasToBridgeOut, _remoteExecutionGas);

// data[0]    ==> flag === 0x01
// data[1:5]  ==> depositNonce
```

3. **`flag == 0x02`**
    - When [encoding the data for the flag 0x02](https://github.com/code-423n4/2023-05-maia/blob/main/src/ulysses-omnichain/BranchBridgeAgent.sol#L680-L692), we can see that **the `depositNonce` is located at the data[1:5]**
```solidity
bytes memory packedData = abi.encodePacked(
    bytes1(0x02),
    depositNonce,
    _dParams.hToken,
    _dParams.token,
    _dParams.amount,
    _normalizeDecimals(_dParams.deposit, ERC20(_dParams.token).decimals()),
    _dParams.toChain,
    _params,
    _gasToBridgeOut,
    _remoteExecutionGas
);

// data[0]    ==> flag === 0x02
// data[1:5]  ==> depositNonce
```

4. **`flag == 0x03`**
    - When [encoding the data for the flag 0x03](https://github.com/code-423n4/2023-05-maia/blob/main/src/ulysses-omnichain/BranchBridgeAgent.sol#L723-L736), we can see that **the `depositNonce` is located at the data[2:6]**
```solidity
bytes memory packedData = abi.encodePacked(
    bytes1(0x03),
    uint8(_dParams.hTokens.length),
    depositNonce,
    _dParams.hTokens,
    _dParams.tokens,
    _dParams.amounts,
    deposits,
    _dParams.toChain,
    _params,
    _gasToBridgeOut,
    _remoteExecutionGas
);

// data[0]    ==> flag === 0x03
// data[1]    ==> hTones.length
// data[2:6]  ==> depositNonce
```

5. **`flag == 0x04`**
    - When [encoding the data for the flag 0x04](https://github.com/code-423n4/2023-05-maia/blob/main/src/ulysses-omnichain/BranchBridgeAgent.sol#L225-L228), we can see that **the `depositNonce` is located at the data[21:25]**
```solidity
bytes memory packedData = abi.encodePacked(
    bytes1(0x04), msg.sender, depositNonce, _params, msg.value.toUint128(), _remoteExecutionGas
);

// data[0]    ==> flag === 0x04
// data[1:21] ==> msg.sender
// data[21:25]  ==> depositNonce
```

6. **`flag == 0x05`**
    - When [encoding the data for the flag 0x05](https://github.com/code-423n4/2023-05-maia/blob/main/src/ulysses-omnichain/BranchBridgeAgent.sol#L244-L257), we can see that **the `depositNonce` is located at the data[21:25]**
```solidity
bytes memory packedData = abi.encodePacked(
    bytes1(0x05),
    msg.sender,
    depositNonce,
    _dParams.hToken,
    _dParams.token,
    _dParams.amount,
    _normalizeDecimals(_dParams.deposit, ERC20(_dParams.token).decimals()),
    _dParams.toChain,
    _params,
    msg.value.toUint128(),
    _remoteExecutionGas
);

// data[0]    ==> flag === 0x05
// data[1:21] ==> msg.sender
// data[21:25]  ==> depositNonce
```

7. **`flag == 0x06`**
    - When [encoding the data for the flag 0x06](https://github.com/code-423n4/2023-05-maia/blob/main/src/ulysses-omnichain/BranchBridgeAgent.sol#L287-L301), we can see that **the `depositNonce` is located at the data[22:26]**
```solidity
bytes memory packedData = abi.encodePacked(
    bytes1(0x06),
    msg.sender,
    uint8(_dParams.hTokens.length),
    depositNonce,
    _dParams.hTokens,
    _dParams.tokens,
    _dParams.amounts,
    _deposits,
    _dParams.toChain,
    _params,
    msg.value.toUint128(),
    _remoteExecutionGas
);

// data[0]     ==> flag === 0x06
// data[1:21]  ==> msg.sender
// data[21]    ==> hTokens.length
// data[22:26] ==> depositNonce
```

- At this point now we know the exact offset where the `depositNonce` is located at for all the possible deposit options.
- Now is time to analyze the offset that are been read depending on the flag in the anyFallback() and validate that the correct offset is been read.

1. For `flags 0x00 , 0x01 & 0x02`, the `depositNonce` is been read from the offset `data[PARAMS_START:PARAMS_TKN_START]`, which is the same as `data[1:5]` ([PARAMS_START == 1](https://github.com/code-423n4/2023-05-maia/blob/main/src/ulysses-omnichain/BranchBridgeAgent.sol#L68) & [PARAMS_TKN_START == 5](https://github.com/code-423n4/2023-05-maia/blob/main/src/ulysses-omnichain/BranchBridgeAgent.sol#L78)), **these 3 flags read correctly the `depositNonce`**

2. For `flag 0x03`, the `depositNonce` is been read from the offset `data[PARAMS_START + PARAMS_START:PARAMS_TKN_START + PARAMS_START]`, which is the same as `data[2:6]` ([PARAMS_START == 1](https://github.com/code-423n4/2023-05-maia/blob/main/src/ulysses-omnichain/BranchBridgeAgent.sol#L68) & [PARAMS_TKN_START == 5](https://github.com/code-423n4/2023-05-maia/blob/main/src/ulysses-omnichain/BranchBridgeAgent.sol#L78)), **this flag also reads correctly the `depositNonce`**

3. For `flag 0x04 & 0x05`, the `depositNonce` is been read from the offset `data[PARAMS_START_SIGNED:PARAMS_START_SIGNED + PARAMS_TKN_START]`, which is the same as `data[21:26]` ([PARAMS_START_SIGNED == 21](https://github.com/code-423n4/2023-05-maia/blob/main/src/ulysses-omnichain/BranchBridgeAgent.sol#L70) & [PARAMS_TKN_START == 5](https://github.com/code-423n4/2023-05-maia/blob/main/src/ulysses-omnichain/BranchBridgeAgent.sol#L78)), **these flags are reading INCORRECTLY the `depositNonce`**, from the above analyzis to detect where the `depositNonce` is located at, we got that **for flags 0x04 & 0x05, the `depositNonce` is located at the offset data[21:25]**

**PoC to demonstrate the correct offset of the depositNonce when data is encoded similar to how flags 0x04 & 0x05 encode it (See the above analysis for more details)**
1. call the `generateData()` function and copy+paste the generated bytes on the rest of the functions
2. Notice how the `readNonce()` returns the correct value of the nonce, and is reading the offset `data[21:25]`
```solidity
pragma solidity 0.8.18;

contract offset {
    uint32 nonce = 3;

    function generateData() external view returns (bytes memory) {
        bytes memory packedData = abi.encodePacked(
            bytes1(0x01),
            msg.sender,
            nonce
        );
        return packedData;
    }

    function readFlag(bytes calldata data) external view returns(bytes1) {
        return data[0];
    }

    function readMsgSender(bytes calldata data) external view returns (address) {
        return address(uint160(bytes20(data[1:21])));
    }

    function readNonce(bytes calldata data) external view returns (uint32) {
        return uint32(
            bytes4(data[21:25])
        );
    }   

}
```

3. For `flag 0x06`, the `depositNonce` is been read from the offset `data[PARAMS_START_SIGNED + PARAMS_START:PARAMS_START_SIGNED + PARAMS_TKN_START + PARAMS_START]`, which is the same as `data[22:27]` ([PARAMS_START_SIGNED == 21](https://github.com/code-423n4/2023-05-maia/blob/main/src/ulysses-omnichain/BranchBridgeAgent.sol#L70), [PARAMS_START == 1](https://github.com/code-423n4/2023-05-maia/blob/main/src/ulysses-omnichain/BranchBridgeAgent.sol#L68) & [PARAMS_TKN_START == 5](https://github.com/code-423n4/2023-05-maia/blob/main/src/ulysses-omnichain/BranchBridgeAgent.sol#L78)), **this flag is also reading INCORRECTLY the `depositNonce`**, from the above analyzis to detect where the `depositNonce` is located at, we got that **for flag 0x06, the `depositNonce` is located at the offset data[22:26]**

**PoC to demonstrate the correct offset of the depositNonce when data is encoded similar to how flag 0x06 encode it (See the above analysis for more details)**
1. call the `generateData()` function and copy+paste the generated bytes on the rest of the functions
2. Notice how the `readNonce()` returns the correct value of the nonce, and is reading the offset `data[22:26]`
```solidity
pragma solidity 0.8.18;

contract offset {
    uint32 nonce = 3;

    function generateData() external view returns (bytes memory) {
        bytes memory packedData = abi.encodePacked(
            bytes1(0x01),
            msg.sender,
            uint8(1),
            nonce
        );
        return packedData;
    }

    function readFlag(bytes calldata data) external view returns(bytes1) {
        return data[0];
    }

    function readMsgSender(bytes calldata data) external view returns (address) {
        return address(uint160(bytes20(data[1:21])));
    }

    function readThirdParameter(bytes calldata data) external view returns(uint8) {
        return uint8(bytes1(data[21]));
    }

    function readNonce(bytes calldata data) external view returns (uint32) {
        return uint32(
            bytes4(data[22:26])
        );
    }

}
```

## Tools Used
Manual Audit

## Recommended Mitigation Steps
- Make sure to read the `depositNonce` from the correct offset, depending on the flag it will be the offset where `depositNonce` is located at:

- For `flags 0x04 & 0x05`, read the offset as follows, either of the two options are correct:
  - `depositNonce` is located at: `data[21:25]`
```solidity
_depositNonce = uint32(bytes4(data[PARAMS_START_SIGNED : PARAMS_START_SIGNED]));
_depositNonce = uint32(bytes4(data[21:25]));
```

- For `flag 0x06`, read the offset as follows, either of the two options are correct:
  - `depositNonce` is located at: `data[22:26]`
```solidity
_depositNonce = uint32(bytes4(data[PARAMS_START_SIGNED + PARAMS_START : PARAMS_START_SIGNED + PARAMS_TKN_START]));
_depositNonce = uint32(bytes4(data[22:26]));
```


## Assessed type

en/de-code