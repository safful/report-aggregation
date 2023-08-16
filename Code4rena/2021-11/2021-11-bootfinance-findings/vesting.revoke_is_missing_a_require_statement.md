## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [Vesting.revoke is missing a require statement](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/116) 

# Handle

Reigada


# Vulnerability details

## Impact
In the Vesting contract, the function revoke() sends the vested tokens to the beneficiary and the remaining tokens that are not vested yet are sent to the multisig address. It makes no sense to allow calling this function once the address has already vested the 100% of the tokens (after 1 year in this case -> uint256 _unlockTimestamp = block.timestamp.add(unixYear);).

Basically in this case the function revoke() would behave like a claim() function but doing some extra checks which waste gas (gas paid by the owner of the contract instead of the beneficiary address) and also emitting an extra event -> https://github.com/code-423n4/2021-11-bootfinance/blob/main/vesting/contracts/Vesting.sol#L123

For that reason, it is recommended to add a require statement that handles this case:

uint256 index = timelocks[_addr].length - 1;
require (block.timestamp < timelocks[_addr][index].releaseTimestamp, 'Account fully vested');

## Tools Used
Manual testing

## Recommended Mitigation Steps
It is recommended to add a require statement that handles this case in the Vesting.revoke() function:

uint256 index = timelocks[_addr].length - 1;
require (block.timestamp < timelocks[_addr][index].releaseTimestamp, 'Account fully vested');

