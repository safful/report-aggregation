## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [`NestedFactory` does not track operators properly](https://github.com/code-423n4/2022-02-nested-findings/issues/38) 

# Lines of code

https://github.com/code-423n4/2022-02-nested/blob/fe6f9ef7783c3c84798c8ab5fc58085a55cebcfc/contracts/NestedFactory.sol#L99-L108
https://github.com/code-423n4/2022-02-nested/blob/fe6f9ef7783c3c84798c8ab5fc58085a55cebcfc/contracts/abstracts/MixinOperatorResolver.sol#L30-L47
https://github.com/code-423n4/2022-02-nested/blob/fe6f9ef7783c3c84798c8ab5fc58085a55cebcfc/contracts/NestedFactory.sol#L110-L122
https://github.com/code-423n4/2022-02-nested/blob/fe6f9ef7783c3c84798c8ab5fc58085a55cebcfc/contracts/abstracts/MixinOperatorResolver.sol#L49-L55


# Vulnerability details

`NestedFactory` extends the `MixinOperatorResolver` contract which comes from the [`synthetix/MixinResolver.sol`](https://github.com/Synthetixio/synthetix/blob/a1786e5d64b5b51212785ade6d8b42435f69c387/contracts/MixinResolver.sol) code base where the expectation is that `isResolverCached()` returns false until [`rebuildCache()` is called and the cache is fully up to date](https://github.com/Synthetixio/synthetix/blob/a1786e5d64b5b51212785ade6d8b42435f69c387/test/contracts/MixinResolver.js#L82-L105). Due to [a medium issue](https://github.com/code-423n4/2021-11-nested-findings/issues/217) identified in a prior contest, the `OperatorResolver.importOperators()` step was made to be atomically combined with the `NestedFactory.rebuildCache()` step. However, the atomicity was not applied everywhere and the ability to add/remove operators from the `NestedFactory` also had other cache-inconsistency issues. There are *four separate instances* of operator tracking problems in this submission.

## Impact
As with the prior issue, many core operations (such as `NestedFactory.create()` and `NestedFactory.swapTokenForTokens()`) are dependant on the assumption that the `operatorCache` cache is synced prior to these functions being executed, but this may not necessarily be the case. Unlike the prior issue which was about updates to the resolver not getting reflected in the cache, this issue is about changes to the factory not updating the cache.

## Proof of Concept

### 1. `removeOperator()` does not call `rebuildCache()`
1. `NestedFactory.removeOperator()` is called to remove an operator
2. A user calls `NestedFactory(MixinOperatorResolver).create()` using that operator and succeedes
3. `NestedFactory.rebuildCache()` is called to rebuild cache
This flow is not aware that the cache is not in sync

```solidity
    /// @inheritdoc INestedFactory
    function addOperator(bytes32 operator) external override onlyOwner {
        require(operator != bytes32(""), "NF: INVALID_OPERATOR_NAME");
        bytes32[] memory operatorsCache = operators;
        for (uint256 i = 0; i < operatorsCache.length; i++) {
            require(operatorsCache[i] != operator, "NF: EXISTENT_OPERATOR");
        }
        operators.push(operator);
        emit OperatorAdded(operator);
    }
```
https://github.com/code-423n4/2022-02-nested/blob/fe6f9ef7783c3c84798c8ab5fc58085a55cebcfc/contracts/NestedFactory.sol#L99-L108

### 2. Using both `removeOperator()` and `rebuildCache()` does not prevent `create()` from using the operator
Even if `removeOperator()` calls `rebuildCache()` the function will still not work because `resolverOperatorsRequired()` only keeps track of remaining operators, and `rebuildCache()` currently has no way of knowing that an entry was removed from that array and that a corresponding entry from `operatorCache` needs to be removed too.

```solidity
    /// @notice Rebuild the operatorCache
    function rebuildCache() external {
        bytes32[] memory requiredOperators = resolverOperatorsRequired();
        bytes32 name;
        IOperatorResolver.Operator memory destination;
        // The resolver must call this function whenever it updates its state
        for (uint256 i = 0; i < requiredOperators.length; i++) {
            name = requiredOperators[i];
            // Note: can only be invoked once the resolver has all the targets needed added
            destination = resolver.getOperator(name);
            if (destination.implementation != address(0)) {
                operatorCache[name] = destination;
            } else {
                delete operatorCache[name];
            }
            emit CacheUpdated(name, destination);
        }
    }
```
https://github.com/code-423n4/2022-02-nested/blob/fe6f9ef7783c3c84798c8ab5fc58085a55cebcfc/contracts/abstracts/MixinOperatorResolver.sol#L30-L47

### 3. `addOperator()` does not call `rebuildCache()`
1. `NestedFactory.addOperator()` is called to add an operator
2. A user calls `NestedFactory(MixinOperatorResolver).create()` using that operator and fails because the operator wasn't in the `resolverOperatorsRequired()` during the last call to `rebuildCaches()`, so the operator isn't in `operatorCache`
3. `NestedFactory.rebuildCache()` is called to rebuild cache
This flow is not aware that the cache is not in sync

```solidity
    /// @inheritdoc INestedFactory
    function removeOperator(bytes32 operator) external override onlyOwner {
        uint256 operatorsLength = operators.length;
        for (uint256 i = 0; i < operatorsLength; i++) {
            if (operators[i] == operator) {
                operators[i] = operators[operatorsLength - 1];
                operators.pop();
                emit OperatorRemoved(operator);
                return;
            }
        }
        revert("NF: NON_EXISTENT_OPERATOR");
    }
```
https://github.com/code-423n4/2022-02-nested/blob/fe6f9ef7783c3c84798c8ab5fc58085a55cebcfc/contracts/NestedFactory.sol#L110-L122

### 4. `isResolverCached()` does not reflect the actual updated-or-not state
This function, like `removeOperator()` is not able to tell that there is an operator that needs to be removed from `resolverCache`, causing the owner not to know a call to `rebuildCache()` is required to 'remove' the operator
```solidity
    /// @notice Check the state of operatorCache
    function isResolverCached() external view returns (bool) {
        bytes32[] memory requiredOperators = resolverOperatorsRequired();
        bytes32 name;
        IOperatorResolver.Operator memory cacheTmp;
        IOperatorResolver.Operator memory actualValue;
        for (uint256 i = 0; i < requiredOperators.length; i++) {
```
https://github.com/code-423n4/2022-02-nested/blob/fe6f9ef7783c3c84798c8ab5fc58085a55cebcfc/contracts/abstracts/MixinOperatorResolver.sol#L49-L55

## Tools Used
Code inspection

## Recommended Mitigation Steps
Add calls to `rebuildCache()` in `addOperator()` and `removeOperator()`, have `INestedFactory` also track operators that have been removed with a new array, and have `isResolverCached()` also check whether this new array is empty or not.


