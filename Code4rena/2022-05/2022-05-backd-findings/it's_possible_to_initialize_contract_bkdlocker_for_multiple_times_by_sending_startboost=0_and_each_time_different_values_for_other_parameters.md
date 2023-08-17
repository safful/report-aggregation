## Tags

- bug
- 2 (Med Risk)
- disagree with severity
- sponsor confirmed

# [it's possible to initialize contract BkdLocker for multiple times by sending startBoost=0 and each time different values for other parameters](https://github.com/code-423n4/2022-05-backd-findings/issues/136) 

# Lines of code

https://github.com/code-423n4/2022-05-backd/blob/2a5664d35cde5b036074edef3c1369b984d10010/protocol/contracts/BkdLocker.sol#L53-L64


# Vulnerability details

## Impact
function `initialize()` of `BkdLocker` suppose to be called one time and contract initialize one time. but if it's called by `startBoost=0` then it's possible to call it again with different values for other parameters. there are some logics based on the values function `initilize()` sets which is in calculating boost and withdraw delay. by initializing multiple times different users get different values for those logics and because rewards are distributed based on boosts so those logics will be wrong too.

## Proof of Concept
This is `initiliaze()` code in `BkdLocker`:
```
    function initialize(
        uint256 startBoost,
        uint256 maxBoost,
        uint256 increasePeriod,
        uint256 withdrawDelay
    ) external override onlyGovernance {
        require(currentUInts256[_START_BOOST] == 0, Error.CONTRACT_INITIALIZED);
        _setConfig(_START_BOOST, startBoost);
        _setConfig(_MAX_BOOST, maxBoost);
        _setConfig(_INCREASE_PERIOD, increasePeriod);
        _setConfig(_WITHDRAW_DELAY, withdrawDelay);
    }
```
As you can see it checks the initialization statue by `currentUInts256[_START_BOOST]`'s value but it's not correct way to do and initializer can set `currentUInts256[_START_BOOST]` value as `0` and set other parameters values and call this function multiple times with different values for `_MAX_BOOST` and `_INCREASE_PERIOD` and `_WITHDRAW_DELAY`. setting different values for those parameters can cause different calculation in `computeNewBoost()` and `prepareUnlock()`. function `computeNewBoost()` is used to calculate users boost parameters which is used on reward distribution. so by changing `_MAX_BOOST` the rewards will be distributed wrongly between old users and new users.

## Tools Used
VIM

## Recommended Mitigation Steps
add some other variable to check the status of initialization of contract. 

