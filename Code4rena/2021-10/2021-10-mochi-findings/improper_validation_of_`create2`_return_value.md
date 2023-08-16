## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Improper Validation Of `create2` Return Value](https://github.com/code-423n4/2021-10-mochi-findings/issues/155) 

# Handle

leastwood


# Vulnerability details

## Impact

The `BeaconProxyDeployer.deploy()` function is used to deploy lightweight proxy contracts that act as each asset's vault. The function does not revert properly if there is a failed contract deployment or revert from the `create2` opcode as it does not properly check the returned address for bytecode. The `create2` opcode returns the expected address which will never be the zero address (as is what is currently checked).

## Proof of Concept

https://github.com/code-423n4/2021-10-mochi/blob/main/projects/mochi-library/contracts/BeaconProxyDeployer.sol#L31

## Tools Used

Manual code review
Discussions with the Mochi team
Discussions with library dev

## Recommended Mitigation Steps

The recommended mitigation was to update `iszero(result)` to `iszero(extcodesize(result))` in the line mentioned above. This change has already been made in the corresponding library which can be found [here](https://github.com/Nipol/bean-contracts/pull/13), however, this needs to also be reflected in Mochi's contracts.

