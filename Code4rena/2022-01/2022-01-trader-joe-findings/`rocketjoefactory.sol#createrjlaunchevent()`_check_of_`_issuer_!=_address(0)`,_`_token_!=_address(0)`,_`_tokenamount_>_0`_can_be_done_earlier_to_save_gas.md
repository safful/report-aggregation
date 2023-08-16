## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [`RocketJoeFactory.sol#createRJLaunchEvent()` Check of `_issuer != address(0)`, `_token != address(0)`, `_tokenAmount > 0` can be done earlier to save gas](https://github.com/code-423n4/2022-01-trader-joe-findings/issues/245) 

# Handle

WatchPug


# Vulnerability details

https://github.com/code-423n4/2022-01-trader-joe/blob/a1579f6453bc4bf9fb0db9c627beaa41135438ed/contracts/RocketJoeFactory.sol#L98-L155

```solidity
    function createRJLaunchEvent(
        address _issuer,
        uint256 _phaseOneStartTime,
        address _token,
        uint256 _tokenAmount,
        uint256 _tokenIncentivesPercent,
        uint256 _floorPrice,
        uint256 _maxWithdrawPenalty,
        uint256 _fixedWithdrawPenalty,
        uint256 _maxAllocation,
        uint256 _userTimelock,
        uint256 _issuerTimelock
    ) external override returns (address) {
        require(
            getRJLaunchEvent[_token] == address(0),
            "RJFactory: token has already been issued"
        );
        require(_issuer != address(0), "RJFactory: issuer can't be 0 address");
        require(_token != address(0), "RJFactory: token can't be 0 address");
        require(_token != wavax, "RJFactory: token can't be wavax");
        require(
            _tokenAmount > 0,
            "RJFactory: token amount needs to be greater than 0"
        );
        require(
            IJoeFactory(factory).getPair(_token, wavax) == address(0) ||
                IJoePair(IJoeFactory(factory).getPair(_token, wavax))
                    .totalSupply() ==
                0,
            "RJFactory: liquid pair already exists"
        );
        // ...
    }
```

`_issuer != address(0)`, `_token != address(0)`, `_tokenAmount > 0` are cheaper than other checks who read storage or do external call. 

Therefore, checking `_issuer != address(0)`, `_token != address(0)`, `_tokenAmount > 0` first can save some gas.


### Recommendation

Change to:

https://github.com/code-423n4/2022-01-trader-joe/blob/a1579f6453bc4bf9fb0db9c627beaa41135438ed/contracts/RocketJoeFactory.sol#L98-L155

```solidity
    function createRJLaunchEvent(
        address _issuer,
        uint256 _phaseOneStartTime,
        address _token,
        uint256 _tokenAmount,
        uint256 _tokenIncentivesPercent,
        uint256 _floorPrice,
        uint256 _maxWithdrawPenalty,
        uint256 _fixedWithdrawPenalty,
        uint256 _maxAllocation,
        uint256 _userTimelock,
        uint256 _issuerTimelock
    ) external override returns (address) {
        require(_issuer != address(0), "RJFactory: issuer can't be 0 address");
        require(_token != address(0), "RJFactory: token can't be 0 address");
        require(
            _tokenAmount != 0,
            "RJFactory: token amount needs to be greater than 0"
        );
        require(
            getRJLaunchEvent[_token] == address(0),
            "RJFactory: token has already been issued"
        );
        require(_token != wavax, "RJFactory: token can't be wavax");
        require(
            IJoeFactory(factory).getPair(_token, wavax) == address(0) ||
                IJoePair(IJoeFactory(factory).getPair(_token, wavax))
                    .totalSupply() ==
                0,
            "RJFactory: liquid pair already exists"
        );
        // ...
    }
```

