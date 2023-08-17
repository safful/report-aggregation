## Tags

- bug
- 3 (High Risk)
- resolved
- sponsor confirmed

# [Existing user’s locked JPEG could be overwritten by new user, causing permanent loss of JPEG funds](https://github.com/code-423n4/2022-04-jpegd-findings/issues/10) 

# Lines of code

https://github.com/code-423n4/2022-04-jpegd/blob/main/contracts/vaults/NFTVault.sol#L375
https://github.com/code-423n4/2022-04-jpegd/blob/main/contracts/lock/JPEGLock.sol#L54-L62


# Vulnerability details

## Details & Impact

A user’s JPEG lock schedule can be overwritten by another user’s if he (the other user) submits and finalizes a proposal to change the same NFT index’s value.

The existing user will be unable to withdraw his locked JPEGs, resulting in permanent lock up of JPEG in the locker contract.

## Proof of Concept

1. `user` successfully proposes and finalizes a proposal to change his NFT’s collateral value
2. Another user (`owner`) does the same for the same NFT index
3. `user` will be unable to withdraw his locked JPEG because schedule has been overwritten

Insert this test case into [`NFTVault.ts`](https://github.com/code-423n4/2022-04-jpegd/blob/main/tests/NFTVault.ts).

```jsx
it.only("will overwrite existing user's JPEG lock schedule", async () => {
  // 0. setup
  const index = 7000;
  await erc721.mint(user.address, index);
  await nftVault
    .connect(dao)
    .setPendingNFTValueETH(index, units(50));
  await jpeg.transfer(user.address, units(150000));
  await jpeg.connect(user).approve(locker.address, units(500000));
  await jpeg.connect(owner).approve(locker.address, units(500000));

  // 1. user has JPEG locked for finalization
  await nftVault.connect(user).finalizePendingNFTValueETH(index);

  // 2. owner submit proposal to further increase NFT value
  await nftVault
    .connect(dao)
    .setPendingNFTValueETH(index, units(100));
  
  // 3. owner finalizes, has JPEG locked
  await nftVault.connect(owner).finalizePendingNFTValueETH(index);

  // user schedule has been overwritten
  let schedule = await locker.positions(index);
  expect(schedule.owner).to.equal(owner.address);

  // user tries to unstake
  // wont be able to because schedule was overwritten
  await timeTravel(days(366));
  await expect(locker.connect(user).unlock(index)).to.be.revertedWith("unauthorized");
});
```

## Recommended Mitigation Steps

1. Release the tokens of the existing schedule. Simple and elegant.

```jsx
// in JPEGLock#lockFor()
LockPosition memory existingPosition = positions[_nftIndex];
if (existingPosition.owner != address(0)) {
  // release jpegs to existing owner
  jpeg.safeTransfer(existingPosition.owner, existingPosition.lockAmount);
}
```

2. Revert in `finalizePendingNFTValueETH()` there is an existing lock schedule. This is less desirable IMO, as there is a use-case for increasing / decreasing the NFT value.

