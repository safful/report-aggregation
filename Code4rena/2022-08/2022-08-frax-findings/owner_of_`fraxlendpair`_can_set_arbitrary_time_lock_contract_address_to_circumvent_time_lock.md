## Tags

- bug
- 2 (Med Risk)
- disagree with severity
- sponsor confirmed

# [Owner of `FraxlendPair` can set arbitrary time lock contract address to circumvent time lock](https://github.com/code-423n4/2022-08-frax-findings/issues/156) 

# Lines of code

https://github.com/code-423n4/2022-08-frax/blob/c4189a3a98b38c8c962c5ea72f1a322fbc2ae45f/src/contracts/FraxlendPair.sol#L206


# Vulnerability details

## Impact

The ownership of a deployed Fraxlend pair is transferred to `COMPTROLLER_ADDRESS` on deployment via `FraxlendPairDeployer_deploySecond`. This very owner is able to change the currently used time lock contract address with the `FraxlendPair.setTimeLock` function. A time lock is enforced on the `FraxlendPair.changeFee` function whenever the protocol fee is adjusted.

However, as the Fraxlend pair owner is able to change the time lock contract address to any other arbitrary (contract) address, it is possible to circumvent this timelock without users knowing. By using a custom smart contract without an enforced time lock, the protocol fee can be changed at any time without a proper time lock.

## Proof of Concept

[FraxlendPair.sol#L206](https://github.com/code-423n4/2022-08-frax/blob/c4189a3a98b38c8c962c5ea72f1a322fbc2ae45f/src/contracts/FraxlendPair.sol#L206)

````solidity
/// @notice The ```setTimeLock``` function sets the TIME_LOCK address
/// @param _newAddress the new time lock address
function setTimeLock(address _newAddress) external onlyOwner {
    emit SetTimeLock(TIME_LOCK_ADDRESS, _newAddress);
    TIME_LOCK_ADDRESS = _newAddress;
}
````

## Tools Used

Manual review

## Recommended mitigation steps

Currently, the owner `COMPTROLLER_ADDRESS` address is trustworthy, however, nothing prevents the above-described scenario. To protect users from sudden protocol fee changes, consider using a minimal time lock implementation directly implemented in the contract without trusting any external contract.
