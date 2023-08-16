## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [RewardDistribution: Optimise _getPid](https://github.com/code-423n4/2021-07-wildcredit-findings/issues/50) 

# Handle

greiart


# Vulnerability details

## Impact

A repeated call to `pidByPairToken[_pair][_token][_isSupply]` can be avoided since it is stored in `poolPosition`. Simply return [poolPosition.pid](http://poolposition.pid). 

## Referenced Codelines

[https://github.com/code-423n4/2021-07-wildcredit/blob/main/contracts/RewardDistribution.sol#L245-L250](https://github.com/code-423n4/2021-07-wildcredit/blob/main/contracts/RewardDistribution.sol#L245-L250)

## Proof of Concept

```jsx
function _getPid(address _pair, address _token, bool _isSupply) internal view returns(uint) {
    PoolPosition memory poolPosition = pidByPairToken[_pair][_token][_isSupply];
    require(poolPosition.added, "RewardDistribution: invalid pool");

    return poolPosition.pid;
}
```

