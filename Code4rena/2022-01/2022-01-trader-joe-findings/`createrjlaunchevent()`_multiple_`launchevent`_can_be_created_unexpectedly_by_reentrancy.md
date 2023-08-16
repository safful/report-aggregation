## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [`createRJLaunchEvent()` Multiple `launchEvent` can be created unexpectedly by reentrancy](https://github.com/code-423n4/2022-01-trader-joe-findings/issues/248) 

# Handle

WatchPug


# Vulnerability details

https://github.com/code-423n4/2022-01-trader-joe/blob/119e12d715ececc31478e833297f124cc15d27c2/contracts/RocketJoeFactory.sol#L97-L154

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

    address launchEvent = Clones.clone(eventImplementation);

    // msg.sender needs to approve RocketJoeFactory
    IERC20(_token).transferFrom(msg.sender, launchEvent, _tokenAmount);

    ILaunchEvent(payable(launchEvent)).initialize(
        _issuer,
        _phaseOneStartTime,
        _token,
        _tokenIncentivesPercent,
        _floorPrice,
        _maxWithdrawPenalty,
        _fixedWithdrawPenalty,
        _maxAllocation,
        _userTimelock,
        _issuerTimelock
    );

    getRJLaunchEvent[_token] = launchEvent;
    isRJLaunchEvent[launchEvent] = true;
    allRJLaunchEvents.push(launchEvent);

    _emitLaunchedEvent(_issuer, _token, _phaseOneStartTime);

    return launchEvent;
}
```

At L132, `_token.transferFrom()` can be used to re-enter the `createRJLaunchEvent()` function, before the storage change at L147-149.

This will allow the attacker to create multiple `launchEvent` contracts and get them listed in `allRJLaunchEvents`.

Even though there is no significant impact as far as we can tell from the smart contract code. We believe this is still unexpected and may cause other parts of the system, say the frontend to malfunction in some cases.

### Recommendation

Consider moving L132 `_token.transferFrom()` to after L147-149 to prevent re-entrance.

