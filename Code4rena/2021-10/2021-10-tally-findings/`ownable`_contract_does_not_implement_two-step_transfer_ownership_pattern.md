## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [`Ownable` Contract Does Not Implement Two-Step Transfer Ownership Pattern](https://github.com/code-423n4/2021-10-tally-findings/issues/78) 

# Handle

leastwood


# Vulnerability details

## Impact

`Swap.sol` inherits OpenZeppelin's `Ownable` contract which enables the `onlyOwner` role to transfer ownership to another address. It's possible that the `onlyOwner` role mistakenly transfers ownership to the wrong address, resulting in a loss of the `onlyOwner` role.

## Proof of Concept

https://github.com/code-423n4/2021-10-tally/blob/main/contracts/governance/EmergencyGovernable.sol
https://github.com/code-423n4/2021-10-tally/blob/main/contracts/swap/Swap.sol

## Tools Used

Manual code review.

## Recommended Mitigation Steps

Consider overriding the `transferOwnership()` function to first nominate an address as the pending owner and implementing an `acceptOwnership()` function which is called by the pending owner to confirm the transfer. Alternatively, as the `onlyOwner` role is not used throughout the contract, it may be useful to remove this contract entirely from the `EmergencyGovernable.sol` contract.

