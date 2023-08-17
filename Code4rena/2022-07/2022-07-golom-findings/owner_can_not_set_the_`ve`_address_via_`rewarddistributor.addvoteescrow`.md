## Tags

- bug
- 3 (High Risk)
- resolved
- sponsor confirmed
- selected-for-report

# [Owner can not set the `ve` address via `RewardDistributor.addVoteEscrow`](https://github.com/code-423n4/2022-07-golom-findings/issues/611) 

# Lines of code

https://github.com/code-423n4/2022-07-golom/blob/e5efa8f9d6dda92a90b8b2c4902320acf0c26816/contracts/rewards/RewardDistributor.sol#L300
https://github.com/code-423n4/2022-07-golom/blob/e5efa8f9d6dda92a90b8b2c4902320acf0c26816/contracts/rewards/RewardDistributor.sol#L173


# Vulnerability details

## Impact

On the initial `RewardDistributor.addVoteEscrow` call, the owner of the contract can set the `ve` address without a timelock (which is as intended according to the function documentation). However, as the function parameter `_voteEscrow` is not used for the assignment, instead the storage variable `pendingVoteEscrow` (which is not initialized, hence `address(0)`) is used, the `ve` storage variable can not be set to the provided `_voteEscrow` address.

This prevents setting the `ve` address (`ve` is set to `address(0)`) and therefore prevents `veNFT` holders to claim reward tokens and Ether rewards via `RewardDistributor.multiStakerClaim`.

## Proof of Concept

[RewardDistributor.sol#L300](https://github.com/code-423n4/2022-07-golom/blob/e5efa8f9d6dda92a90b8b2c4902320acf0c26816/contracts/rewards/RewardDistributor.sol#L300)

```solidity
function addVoteEscrow(address _voteEscrow) external onlyOwner {
    if (address(ve) == address(0)) {
        ve = VE(pendingVoteEscrow); // @audit-info The wrong variable is used. It should be `_voteEscrow`
    } else {
        voteEscrowEnableDate = block.timestamp + 1 days;
        pendingVoteEscrow = _voteEscrow;
    }
}
```

[RewardDistributor.sol#L173](https://github.com/code-423n4/2022-07-golom/blob/e5efa8f9d6dda92a90b8b2c4902320acf0c26816/contracts/rewards/RewardDistributor.sol#L173)

```solidity
function multiStakerClaim(uint256[] memory tokenids, uint256[] memory epochs) public {
    require(address(ve) != address(0), ' VE not added yet'); // @audit-info reverts if `ve` is not initialized

    ...
}
```

## Tools Used

Manual review

## Recommended mitigation steps

Use the correct function parameter `_voteEscrow`:

```solidity
function addVoteEscrow(address _voteEscrow) external onlyOwner {
    if (address(ve) == address(0)) {
        ve = VE(_voteEscrow);
    } else {
        voteEscrowEnableDate = block.timestamp + 1 days;
        pendingVoteEscrow = _voteEscrow;
    }
}
```
