## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [At `Alchemist.sol#acceptGovernance`, cache `pendingGovernance` earlier to save gas](https://github.com/code-423n4/2021-11-yaxis-findings/issues/23) 

# Handle

0x0x0x


# Vulnerability details

## Impact
`Alchemist.sol#acceptGovernance`(L216-225) is:
```
    function acceptGovernance() external {
        require(msg.sender == _pendingGovernance, 'sender is not pendingGovernance');
        address _pendingGovernance = pendingGovernance;
        governance = _pendingGovernance;

        emit GovernanceUpdated(_pendingGovernance);
    }
```
It can be replaced with following code to save gas:
```
    function acceptGovernance() external {
        address _pendingGovernance = pendingGovernance;
        require(msg.sender == _pendingGovernance, 'sender is not pendingGovernance');
        governance = _pendingGovernance;

        emit GovernanceUpdated(_pendingGovernance);
    }
```
## Tools Used

Manual analysis

