## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed

# [`Adminable::setupAdmin` uses deprecated function](https://github.com/code-423n4/2021-10-slingshot-findings/issues/50) 

# Handle

pmerkleplant


# Vulnerability details

The `setupAdmin` function in `Adminable.sol` uses the `_setupRole` function
from OpenZeppelin's `AccessControl.sol`.

This function is marked as deprecated in favor of `AccessControl::_grantRole`.

See [line 21 in Adminable.sol](https://github.com/code-423n4/2021-10-slingshot/blob/main/contracts/Adminable.sol#L21)
and [line 183 in OpenZeppelin's AccessControl.sol](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/access/AccessControl.sol#L183)

