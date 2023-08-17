## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [FraxlendPair#setTimeLock: Allows the owner to reset TIME_LOCK_ADDRESS](https://github.com/code-423n4/2022-08-frax-findings/issues/129) 

# Lines of code

https://github.com/code-423n4/2022-08-frax/blob/c4189a3a98b38c8c962c5ea72f1a322fbc2ae45f/src/contracts/FraxlendPairCore.sol#L84-L86
https://github.com/code-423n4/2022-08-frax/blob/c4189a3a98b38c8c962c5ea72f1a322fbc2ae45f/src/contracts/FraxlendPair.sol#L204-L207


# Vulnerability details

## Impact
Allows to reset **TIME_LOCK_ADDRESS** value multiple times by the owner. According to comments in FraxlendPairCore this should act as a constant/immutable value. Given that this value will be define through function **setTimeLock** in **FraxLendPair** contract this value can changed whenever the owner wants. This does not seem the expected behaviour.

## Proof of Concept
The owner can call whenever they want the function **setTimeLock**, which reset the value of **TIME_LOCK_ADDRESS**

## Tools Used
Manual read

## Recommended Mitigation Steps
Add a bool which act as mutex if **TIME_LOCK_ADDRESS** has already been set, and modify **setTimeLock** function in FraxlendPair contract
```solidity
// In FraxlendPair contract
bool public timelockSetted;
function setTimeLock(address _newAddress) external onlyOwner {
        require(!timelockSetted);
        emit SetTimeLock(TIME_LOCK_ADDRESS, _newAddress);
        TIME_LOCK_ADDRESS = _newAddress;
        timelockeSetted=true;
}
```