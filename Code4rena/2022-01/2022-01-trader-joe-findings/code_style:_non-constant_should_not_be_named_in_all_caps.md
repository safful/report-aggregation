## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed

# [Code Style: non-constant should not be named in all caps](https://github.com/code-423n4/2022-01-trader-joe-findings/issues/230) 

# Handle

WatchPug


# Vulnerability details

Non-constant (especially public) variables should not be in `SCREAMING_SNAKE_CASE`, or they may be misunderstood as constants.

Consider changing to `camelCase`.

See: https://docs.soliditylang.org/en/v0.8.11/style-guide.html?highlight=name#local-and-state-variable-names

Instances include:

https://github.com/code-423n4/2022-01-trader-joe/blob/a1579f6453bc4bf9fb0db9c627beaa41135438ed/contracts/interfaces/IRocketJoeFactory.sol#L35-L39

```solidity
function PHASE_ONE_DURATION() external view returns (uint256);

function PHASE_ONE_NO_FEE_DURATION() external view returns (uint256);

function PHASE_TWO_DURATION() external view returns (uint256);
```

https://github.com/code-423n4/2022-01-trader-joe/blob/a1579f6453bc4bf9fb0db9c627beaa41135438ed/contracts/RocketJoeFactory.sol#L29-L31

```solidity
uint256 public override PHASE_ONE_DURATION = 2 days;
uint256 public override PHASE_ONE_NO_FEE_DURATION = 1 days;
uint256 public override PHASE_TWO_DURATION = 1 days;
```

https://github.com/code-423n4/2022-01-trader-joe/blob/a1579f6453bc4bf9fb0db9c627beaa41135438ed/contracts/RocketJoeFactory.sol#L200-L214

```solidity
function setPhaseDuration(uint256 _phaseNumber, uint256 _duration)
    external
    override
    onlyOwner
{
    if (_phaseNumber == 1) {
        require(
            _duration > PHASE_ONE_NO_FEE_DURATION,
            "RJFactory: phase one duration lower than no fee duration"
        );
        PHASE_ONE_DURATION = _duration;
    } else if (_phaseNumber == 2) {
        PHASE_TWO_DURATION = _duration;
    }
}
```

https://github.com/code-423n4/2022-01-trader-joe/blob/a1579f6453bc4bf9fb0db9c627beaa41135438ed/contracts/RocketJoeFactory.sol#L218-L228

```solidity
function setPhaseOneNoFeeDuration(uint256 _noFeeDuration)
    external
    override
    onlyOwner
{
    require(
        _noFeeDuration < PHASE_ONE_DURATION,
        "RJFactory: no fee duration bigger than phase one duration"
    );
    PHASE_ONE_NO_FEE_DURATION = _noFeeDuration;
}
```

