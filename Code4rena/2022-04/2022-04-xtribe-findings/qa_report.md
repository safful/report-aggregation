## Tags

- bug
- sponsor confirmed
- QA (Quality Assurance)

# [QA Report](https://github.com/code-423n4/2022-04-xtribe-findings/issues/76) 

# QA Report

## Table of Contents

- [summary](#summary)
- [Typos](#typos)
- [Comment Missing function parameter](#comment-missing-function-parameter)
- [Function missing comments](#function-missing-comments)
- [Setters should emit an event](#setters-should-emit-an-event)
- [Setters should check the input value](#setters-should-check-the-input-value)
- [assert statement should not be used](#assert-statement-should-not-be-used)



# summary

> Few vulnerabilities were found examining the contracts. The main concerns are with the presence of two `assert` statements, which is bad practice.
>
> Setters should check the input value before updating a storage variable.


# Typos

## PROBLEM

There are a few typos in the contracts.



## SEVERITY

Non-Critical




## PROOF OF CONCEPT

Instances include:


### FlywheelCore.sol

```
FlywheelCore.sol:97: //user should be secondUser
```



## TOOLS USED

Manual Analysis



## MITIGATION

Correct the typos.


# Comment Missing function parameter

## PROBLEM

Some of the function comments are missing function parameters or returns



## SEVERITY

Non-Critical




## PROOF OF CONCEPT

Instances include:


### xTRIBE.sol

```
xTRIBE.sol:89 address[] calldata accounts
```

### FlywheelGaugeRewards.sol

```
FlywheelGaugeRewards.sol:133: uint256 numRewards
```

### ERC20Gauges.sol

```
ERC20Gauges.sol:97: address gauge
ERC20Gauges.sol:102: address gauge
ERC20Gauges.sol:143: address gauge
ERC20Gauges.sol:163: address user
ERC20Gauges.sol:168: address user, address gauge
ERC20Gauges.sol:193: address user
ERC20Gauges.sol:198: address user
ERC20Gauges.sol:495: address oldGauge, address newGauge
ERC20Gauges.sol:510: address account, bool canExceedMax
```

### ERC20MultiVotes.sol

```
ERC20MultiVotes.sol:36: address account, uint32 pos
ERC20MultiVotes.sol:41: address account
ERC20MultiVotes.sol:114: uint256 newMax
ERC20MultiVotes.sol:122: address account, bool canExceedMax
```


## TOOLS USED

Manual Analysis



## MITIGATION

Add a comment for these parameters


# Function missing comments

## PROBLEM

Some functions are missing comments.



## SEVERITY

Non-Critical




## PROOF OF CONCEPT

Instances include:


### FlywheelCore.sol

```
FlywheelCore.sol:146: _addStrategyForRewards(ERC20 strategy)
FlywheelCore.sol:154: getAllStrategies()
```

### FlywheelGaugeRewards.sol

```
FlywheelGaugeRewards.sol:179:_queueRewards(
        address[] memory gauges,
        uint32 currentCycle,
        uint32 lastCycle,
        uint256 totalQueuedForCycle
    )
```

### ERC20Gauges.sol

```
ERC20Gauges.sol:251: _incrementGaugeWeight(
        address user,
        address gauge,
        uint112 weight,
        uint32 cycle
    )
ERC20Gauges.sol:273: _incrementUserAndGlobalWeights(
        address user,
        uint112 weight,
        uint32 cycle
    )
ERC20Gauges.sol:334: _decrementGaugeWeight(
        address user,
        address gauge,
        uint112 weight,
        uint32 cycle
    ) 
ERC20Gauges.sol:353: _decrementUserAndGlobalWeights(
        address user,
        uint112 weight,
        uint32 cycle
    )
ERC20Gauges.sol:457: _addGauge(address gauge)
ERC20Gauges.sol:479: _removeGauge(address gauge)
```

### ERC20MultiVotes.sol

```
ERC20MultiVotes.sol:236: _incrementDelegation(
        address delegator,
        address delegatee,
        uint256 amount
    )
ERC20MultiVotes.sol:258: _undelegate(
        address delegator,
        address delegatee,
        uint256 amount
    )
ERC20MultiVotes.sol:276: _writeCheckpoint(
        address delegatee,
        function(uint256, uint256)
```

## TOOLS USED

Manual Analysis



## MITIGATION

Add comments to these functions

# Setters should emit an event

## PROBLEM

All setters should emit an event, so the Dapps can detect important changes



## SEVERITY

Non-Critical




## PROOF OF CONCEPT

Instances include:


### FlywheelGaugeRewards.sol

```
FlywheelGaugeRewards.sol:273:setRewardsStream(IRewardsStream newRewardsStream) 
```

## TOOLS USED

Manual Analysis



## MITIGATION

Add the following event to the contract, and emit it at the end of the function
```
event RewardsStreamUpdated(IRewardsStream newRewardsStream);
```

# Setters should check the input value

## PROBLEM

Setters should check the input value - ie make revert if it is the zero address or zero



## SEVERITY

Low




## PROOF OF CONCEPT

Instances include:


### FlywheelCore.sol

```
FlywheelCore.sol:165:setFlywheelRewards(IFlywheelRewards newFlywheelRewards)
FlywheelCore.sol:183:setBooster(IFlywheelBooster newBooster)
```

### FlywheelGaugeRewards.sol

```
FlywheelGaugeRewards.sol:273:setRewardsStream(IRewardsStream newRewardsStream)
```

### ERC20Gauges.sol

```
ERC20Gauges.sol:502:setMaxGauges(uint256 newMax)
```

### ERC20MultiVotes.sol

```
ERC20MultiVotes.sol:502:setMaxDelegates(uint256 newMax)
```

## TOOLS USED

Manual Analysis



## MITIGATION

Add non-zero checks


# assert statement should not be used

## IMPACT

Properly functioning code should never reach a failing assert statement. If it happened, it would indicate the presence of a bug in the contract. A failing assert uses all the remaining gas, which can be financially painful for a user.


## SEVERITY

Low



## PROOF OF CONCEPT

Instances include:

### FlywheelGaugeRewards.sol

```
FlywheelGaugeRewards.sol:196: assert(queuedRewards.storedCycle == 0 || queuedRewards.storedCycle >= lastCycle);
FlywheelGaugeRewards.sol:235: assert(queuedRewards.storedCycle >= cycle);
```

## TOOLS USED

Manual Analysis



## MITIGATION

Replace the assert statement with a require statement or a custom error


