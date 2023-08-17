## Tags

- bug
- 3 (High Risk)
- primary issue
- sponsor confirmed
- selected for report
- H-03

# [Wrong implementation of function `LBPair.setFeeParameter` can break the funcionality of LBPair and make user's tokens locked ](https://github.com/code-423n4/2022-10-traderjoe-findings/issues/384) 

# Lines of code

https://github.com/code-423n4/2022-10-traderjoe/blob/79f25d48b907f9d0379dd803fc2abc9c5f57db93/src/LBPair.sol#L905-L917


# Vulnerability details

## Vulnerable detail 
Struct `FeeParameters` contains 12 fields as follows: 
```solidity=
struct FeeParameters {
    // 144 lowest bits in slot 
    uint16 binStep;
    uint16 baseFactor;
    uint16 filterPeriod; 
    uint16 decayPeriod; 
    uint16 reductionFactor; 
    uint24 variableFeeControl;
    uint16 protocolShare;
    uint24 maxVolatilityAccumulated; 
    
    // 112 highest bits in slot 
    uint24 volatilityAccumulated;
    uint24 volatilityReference;
    uint24 indexRef;
    uint40 time; 
}
```
Function [`LBPair.setFeeParamters(bytes _packedFeeParamters)`](https://github.com/code-423n4/2022-10-traderjoe/blob/79f25d48b907f9d0379dd803fc2abc9c5f57db93/src/LBPair.sol#L788-L790) is used to set the first 8 fields which was stored in 144 lowest bits of `LBPair._feeParameter`'s slot to 144 lowest bits of `_packedFeeParameters` (The layout of `_packedFeeParameters` can be seen [here](https://github.com/code-423n4/2022-10-traderjoe/blob/79f25d48b907f9d0379dd803fc2abc9c5f57db93/src/LBFactory.sol#L572-L584)).
```solidity=
/// url = https://github.com/code-423n4/2022-10-traderjoe/blob/79f25d48b907f9d0379dd803fc2abc9c5f57db93/src/LBPair.sol#L905-L917

/// @notice Internal function to set the fee parameters of the pair
/// @param _packedFeeParameters The packed fee parameters
function _setFeesParameters(bytes32 _packedFeeParameters) internal {
    bytes32 _feeStorageSlot;
    assembly {
        _feeStorageSlot := sload(_feeParameters.slot)
    }

    /// [#explain]  it will get 112 highest bits of feeStorageSlot,
    ///             and stores it in the 112 lowest bits of _varParameters 
    uint256 _varParameters 
        = _feeStorageSlot.decode(type(uint112).max, _OFFSET_VARIABLE_FEE_PARAMETERS/*=144*/);

    /// [#explain]  get 144 lowest bits of packedFeeParameters 
    ///             and stores it in the 144 lowest bits of _newFeeParameters  
    uint256 _newFeeParameters = _packedFeeParameters.decode(type(uint144).max, 0);

    assembly {
        // [$audit-high] wrong operation `or` here 
        //              Mitigate: or(_newFeeParameters, _varParameters << 144)    
        sstore(_feeParameters.slot, or(_newFeeParameters, _varParameters))
    }
}
```
As we can see in the implementation of `LBPair._setFeesParametes` above, it gets the 112 highest bits of `_feeStorageSlot` and stores it in the 112 lowest bits of `_varParameter`. Then it gets the 144 lowest bits of `packedFeeParameter` and stores it in the 144 lowest bits of `_newFeeParameters`. 

Following the purpose of function `setFeeParameters`, the new `LBPair._feeParameters` should form as follow: 
```
// keep 112 highest bits remain unchanged 
// set 144 lowest bits to `_newFeeParameter`
[...112 bits...][....144 bits.....]
[_varParameters][_newFeeParameters]
```
It will make `feeParameters = _newFeeParameters | (_varParameters << 144)`. But current implementation just stores the `or` value of `_varParameters` and `_newFeeParameter` into `_feeParameters.slot`. It forgot to shift left the `_varParameters` 144 bits before executing `or` operation. 

This will make the value of `binStep`, ..., `maxVolatilityAccumulated` incorrect, and also remove the value (make the bit equal to 0) of `volatilityAccumulated`, ..., `time`.

## Impact
* Incorrect fee calculation when executing an action with LBPair (swap, flashLoan, mint)
* Break the functionality of LBPair. The user can't swap/mint/flashLoan
--> Make all the tokens stuck in the pools 

## Proof of concept 
Here is our test script to describe the impacts 
* https://gist.github.com/WelToHackerLand/012e44bb85420fb53eb0bbb7f0f13769

You can place this file into `/test` folder and run it using 
```bash=
forge test --match-contract High1Test -vv
```

Explanation of test script:
1. First we create a pair with `binStep = DEFAULT_BIN_STEP = 25`
2. We do some actions (add liquidity -> mint -> swap) to increase the value of `volatilityAccumulated` from `0` to `60000`
3. We call function `factory.setFeeParametersOnPair` to set new fee parameters. 
4. After that the value of `volatilityAccumulated` changed to value `0` (It should still be unchanged after `factory.setFeeParametersOnPair`) 
5. We check the value of `binStep` and it changed from`25` to `60025` 
    * `binStep` has that value because [line 915](https://github.com/code-423n4/2022-10-traderjoe/blob/79f25d48b907f9d0379dd803fc2abc9c5f57db93/src/LBPair.sol#L915) set `binStep = uint16(volatilityAccumulated) | binStep = 60000 | 25 = 60025`. 
6. This change of `binStep` value will break all the functionality of `LBPair` cause `binStep > Constant.BASIS_POINT_MAX = 10000` 
--> `Error: BinStepOverflows` 


## Tools Used
Foundry 
 
## Recommended Mitigation Steps
Modify function `LBPair._setFeesParaters` as follow: 
```solidity=
/// url = https://github.com/code-423n4/2022-10-traderjoe/blob/79f25d48b907f9d0379dd803fc2abc9c5f57db93/src/LBPair.sol#L905-L917
function _setFeesParameters(bytes32 _packedFeeParameters) internal {
    bytes32 _feeStorageSlot;
    assembly {
        _feeStorageSlot := sload(_feeParameters.slot)
    }


    uint256 _varParameters = _feeStorageSlot.decode(type(uint112).max, _OFFSET_VARIABLE_FEE_PARAMETERS);
    uint256 _newFeeParameters = _packedFeeParameters.decode(type(uint144).max, 0);


    assembly {
        sstore(_feeParameters.slot, or(_newFeeParameters, shl(144, _varParameters)))
    }
}
```

