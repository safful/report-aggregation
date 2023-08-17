## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed
- valid

# [Gas Optimizations](https://github.com/code-423n4/2022-06-badger-findings/issues/141) 

## Use bytes32 rather than string/bytes. ~300 gas (with optimization we spend 314 gas less while without it we spend 426 gas)

If you can fit your data in 32 bytes, then you should use bytes32 datatype rather than bytes or strings as it is much cheaper in solidity. Basically, Any fixed size variable in solidity is cheaper than variable size.
[From the docs](https://docs.soliditylang.org/en/v0.5.8/types.html#bytes-and-strings-as-arrays), As a general rule, use bytes for arbitrary-length raw byte data and string for arbitrary-length string (UTF-8) data. If you can limit the length to a certain number of bytes, always use one of the value types bytes1 to bytes32 because they are much cheaper.

File: MyStrategy.sol [line 131-133](https://github.com/Badger-Finance/vested-aura/blob/d504684e4f9b56660a9e6c6dfb839dcebac3c174/contracts/MyStrategy.sol#L131-L133)

```
    function version() external pure returns (string memory) {
        return "1.0";
    }
```

**Tests for the above function**
**Using strings**
21805 gas without optimization
21530 gas with optimization

**Using bytes32**
21379 gas without optimization
21216 gas with optimization

**Gas Estimates with optimization turned on**

```
// 21530 gas
    function version() external pure returns (string memory) {
        return "1.0";
    }
// 21216 gas
      function version() external pure returns (bytes32) {
        return bytes32("1.0");
    }
```

## ++i costs less gas compared to i++ or i += 1  (~5 gas per iteration)

++i costs less gas compared to i++ or i += 1 for unsigned integer, as pre-increment is cheaper (about 5 gas per iteration). This statement is true even with the optimizer enabled.

i++ increments i and returns the initial value of i. Which means:

```
uint i = 1;  
i++; // == 1 but i == 2  
```

But ++i returns the actual incremented value:

```
uint i = 1;  
++i; // == 2 and i == 2 too, so no need for a temporary variable  
```

In the first case, the compiler has to create a temporary variable (when used) for returning 1 instead of 2

Instances include:

File: MyStrategy.sol [line 118](https://github.com/Badger-Finance/vested-aura/blob/d504684e4f9b56660a9e6c6dfb839dcebac3c174/contracts/MyStrategy.sol#L118)

```
        for(uint i = 0; i < length; i++){
```

Similar thing to my proposal was implemented in the following line

File: MyStrategy.sol [line 153](https://github.com/Badger-Finance/vested-aura/blob/d504684e4f9b56660a9e6c6dfb839dcebac3c174/contracts/MyStrategy.sol#L153)

```
        for (uint256 i; i < numRewards; ++i) {
```

**Other instances to modify**
File: MyStrategy.sol [line 300](https://github.com/Badger-Finance/vested-aura/blob/d504684e4f9b56660a9e6c6dfb839dcebac3c174/contracts/MyStrategy.sol#L300)
```
        for (uint256 i = 0; i < _claims.length; i++) {
```

File: MyStrategy.sol [line 317](https://github.com/Badger-Finance/vested-aura/blob/d504684e4f9b56660a9e6c6dfb839dcebac3c174/contracts/MyStrategy.sol#L317)
```
        for (uint256 i = 0; i < _claims.length; i++) {
```


## Splitting require() statements that use && saves gas - 8 gas per &&

Instead of using the && operator in a single require statement to check multiple conditions,using multiple require statements with 1 condition per require statement will save 8 GAS per &&
The gas difference would only be realized if the revert condition is realized(met).

File: MyStrategy.sol [line 184-187](https://github.com/Badger-Finance/vested-aura/blob/d504684e4f9b56660a9e6c6dfb839dcebac3c174/contracts/MyStrategy.sol#L184-L187)

```
        require(
            balanceOfPool() == 0 && LOCKER.balanceOf(address(this)) == 0,
            "You have to wait for unlock or have to manually rebalance out of it"
        );
```

The above should be modified to:

```
  require(balanceOfPool() == 0,"You have to wait for unlock or have to manually rebalance out of it" );
    require(LOCKER.balanceOf(address(this)) == 0,"You have to wait for unlock or have to manually rebalance out of it");
```

**Proof**
**The following tests were carried out in remix with both optimization turned on and off**

```function
    require ( a > 1 && a < 5, "Initialized");
    return  a + 2;
}
```

**Execution cost**
21617 with optimization and using &&
21976 without optimization and using &&

After splitting the require statement

```function
    require (a > 1 ,"Initialized");
    require (a < 5 , "Initialized");
    return a + 2;
}
```

**Execution cost**
21609 with optimization and split require
21968 without optimization and using split require


## Cache the length of arrays in loops ~6 gas per iteration
Reading array length at each iteration of the loop takes 6 gas (3 for mload and 3 to place memory_offset) in the stack.

The solidity compiler will always read the length of the array during each iteration. That is,

   1.if it is a storage array, this is an extra sload operation (100 additional extra gas (EIP-2929 2) for each iteration except for the first),
   2.if it is a memory array, this is an extra mload operation (3 additional gas for each iteration except for the first),
   3.if it is a calldata array, this is an extra calldataload operation (3 additional gas for each iteration except for the first)

This extra costs can be avoided by caching the array length (in stack):
 When reading the length of an array,  **sload** or **mload** or **calldataload** operation is only called once and subsequently replaced by a cheap **dupN** instruction. Even though mload , calldataload and dupN have the same gas cost, mload and calldataload needs an additional dupN to put the offset in the stack, i.e., an extra 3 gas. which brings this to 6 gas
 
Here, I suggest storing the array’s length in a variable before the for-loop, and use it instead:


File: MyStrategy.sol [line 300](https://github.com/Badger-Finance/vested-aura/blob/d504684e4f9b56660a9e6c6dfb839dcebac3c174/contracts/MyStrategy.sol#L300)

```
        for (uint256 i = 0; i < _claims.length; i++) {
```


File: MyStrategy.sol [line 317](https://github.com/Badger-Finance/vested-aura/blob/d504684e4f9b56660a9e6c6dfb839dcebac3c174/contracts/MyStrategy.sol#L317)

```
        for (uint256 i = 0; i < _claims.length; i++) {
```


Something similar to my propasal has been implemented already on [line 153](https://github.com/Badger-Finance/vested-aura/blob/d504684e4f9b56660a9e6c6dfb839dcebac3c174/contracts/MyStrategy.sol#L153)

```
    function balanceOfRewards() external view override returns (TokenAmount[] memory rewards) {
        IAuraLocker.EarnedData[] memory earnedData = LOCKER.claimableRewards(address(this));
        uint256 numRewards = earnedData.length;
        rewards = new TokenAmount[](numRewards);
        for (uint256 i; i < numRewards; ++i) {
            rewards[i] = TokenAmount(earnedData[i].token, earnedData[i].amount);
        }
    }
```

### No need to initialize variables with their default values

If a variable is not set/initialized, it is assumed to have the default value (0, false, 0x0 etc depending on the data type). If you explicitly initialize it with its default value, you are just wasting gas.
It costs more gas to initialize variables to zero than to let the default of zero be applied

File: MyStrategy.sol [line 115-121](https://github.com/Badger-Finance/vested-aura/blob/d504684e4f9b56660a9e6c6dfb839dcebac3c174/contracts/MyStrategy.sol#L115-L121)

```
    /// @dev Bulk function for sweepRewardToken
    function sweepRewards(address[] calldata tokens) external {
        uint256 length = tokens.length;
        for(uint i = 0; i < length; i++){
            sweepRewardToken(tokens[i]);
        }
    }
```

Similar thing was done on the following line:
File:MyStrategy.sol [line 153-155](https://github.com/Badger-Finance/vested-aura/blob/d504684e4f9b56660a9e6c6dfb839dcebac3c174/contracts/MyStrategy.sol#L153-L155)

```
        for (uint256 i; i < numRewards; ++i) {
            rewards[i] = TokenAmount(earnedData[i].token, earnedData[i].amount);
        }
```

**Other instances to modify**
File:MyStrategy.sol [line 317](https://github.com/Badger-Finance/vested-aura/blob/d504684e4f9b56660a9e6c6dfb839dcebac3c174/contracts/MyStrategy.sol#L317)

```
        for (uint256 i = 0; i < _claims.length; i++) {
```

## use shorter revert strings(less than 32 bytes) 

You can (and should) attach error reason strings along with require statements to make it easier to understand why a contract call reverted. These strings, however, take space in the deployed bytecode. Every reason string takes at least 32 bytes so make sure your string fits in 32 bytes or it will become more expensive.

Shortening revert strings to fit in 32 bytes will decrease deployment time gas and will decrease runtime gas when the revert condition is met.

Revert strings that are longer than 32 bytes require at least one additional mstore, along with additional overhead for computing memory offset, etc.


File: MyStrategy.sol [line 184](https://github.com/Badger-Finance/vested-aura/blob/d504684e4f9b56660a9e6c6dfb839dcebac3c174/contracts/MyStrategy.sol#L184-L187)

```
        require(
            balanceOfPool() == 0 && LOCKER.balanceOf(address(this)) == 0,
            "You have to wait for unlock or have to manually rebalance out of it"
        );
```
