## Tags

- bug
- 2 (Med Risk)
- disagree with severity
- resolved
- sponsor confirmed

# [Reenterancy in `_sendSherRewardsToOwner()`](https://github.com/code-423n4/2022-01-sherlock-findings/issues/60) 

# Handle

kirk-baird


# Vulnerability details

## Impact

This is a reentrancy vulnerability that would allow the attacker to drain the entire SHER balance of the contract.

Note: this attack requires gaining control of execution `sher.transfer()` which will depend on the implementation of the SHER token. Control may be gained by the attacker if the contract implements ERC777 or otherwise makes external calls during `transfer()`.

## Proof of Concept

See [_sendSherRewards](https://github.com/code-423n4/2022-01-sherlock/blob/main/contracts/Sherlock.sol#L442)

```solidity
  function _sendSherRewardsToOwner(uint256 _id, address _nftOwner) internal {
    uint256 sherReward = sherRewards_[_id];
    if (sherReward == 0) return;

    // Transfers the SHER tokens associated with this NFT ID to the address of the NFT owner
    sher.safeTransfer(_nftOwner, sherReward);
    // Deletes the SHER reward mapping for this NFT ID
    delete sherRewards_[_id];
  }
```

Here `sherRewards` are deleted after the potential external call is made in `sher.safeTransfer()`. As a result if an attacker reenters this function `sherRewards_` they will still maintain the original balance of rewards and again transfer the SHER tokens.

As `_sendSherRewardsToOwner()` is `internal` the attack can be initiated through the `external` function `ownerRestake()` [see here.](https://github.com/code-423n4/2022-01-sherlock/blob/main/contracts/Sherlock.sol#L595)

Steps to produce the attack:

1) Deploy attack contract to handle reenterancy
2) Call `initialStake()` from the attack contract with the smallest `period`
3) Wait for `period` amount of time to pass
4) Have the attack contract call `ownerRestake()`. The attack contract will gain control of the (See note above about control flow). This will recursively call `ownerRestake()` until the balance of `Sherlock` is 0 or less than the user's reward amount. Then allow reentrancy loop to unwind and complete.

## Tools Used

n/a

## Recommended Mitigation Steps

Reentrancy can be mitigated by one of two solutions.

The first option is to add a reentrancy guard like `nonReentrant` the is used in `SherlockClaimManager.sol`.

The second option is to use the checks-effects-interactions pattern. This would involve doing all validation checks and state changes before making any potential external calls. For example the above function could be modified as follows.

```solidity
  function _sendSherRewardsToOwner(uint256 _id, address _nftOwner) internal {
    uint256 sherReward = sherRewards_[_id];
    if (sherReward == 0) return;

    // Deletes the SHER reward mapping for this NFT ID
    delete sherRewards_[_id];

    // Transfers the SHER tokens associated with this NFT ID to the address of the NFT owner
    sher.safeTransfer(_nftOwner, sherReward);
  }
```

Additionally the following functions are not exploitable however should be updated to use the check-effects-interations pattern.
- `Sherlock._redeemShares()` should do `_transferTokensOut()` last.
- `Sherlock.initialStake()` should do `token.safeTransferFrom(msg.sender, address(this), _amount);` last
- `SherClaim.add()` should do `sher.safeTransferFrom(msg.sender, address(this), _amount);` after updating `userClaims` 
- `SherlockProtocolManager.depositToActiveBalance()` should do `token.safeTransferFrom(msg.sender, address(this), _amount);` after updating `activeBalances`



