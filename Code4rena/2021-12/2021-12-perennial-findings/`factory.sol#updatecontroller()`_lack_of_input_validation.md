## Tags

- bug
- 1 (Low Risk)
- resolved
- sponsor confirmed

# [`Factory.sol#updateController()` Lack of input validation](https://github.com/code-423n4/2021-12-perennial-findings/issues/35) 

# Handle

WatchPug


# Vulnerability details

`newController.owner` should be validated to make sure the new owner's address is not `address(0)`.

Otherwise, if the owner mistakenly calls `updateController()` with improper inputs can result in all the `onlyOwner(controllerId)` methods being unaccessible.

https://github.com/code-423n4/2021-12-perennial/blob/fd7c38823833a51ae0c6ae3856a3d93a7309c0e4/protocol/contracts/factory/Factory.sol#L103-L106

```solidity=103
function updateController(uint256 controllerId, Controller memory newController) onlyOwner(controllerId) external {
    _controllers[controllerId] = newController;
    emit ControllerUpdated(controllerId, newController.owner, newController.treasury);
}
```

