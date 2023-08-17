## Tags

- bug
- 2 (Med Risk)
- high quality report
- sponsor confirmed

# [Inconsistent logic of increase unlock time to the expired locks](https://github.com/code-423n4/2022-08-fiatdao-findings/issues/254) 

# Lines of code

https://github.com/code-423n4/2022-08-fiatdao/blob/main/contracts/VotingEscrow.sol#L493-L523


# Vulnerability details

# [2022-08-fiatdao] Inconsistent logic of increase unlock time to the expired locks
## Impact
Can not prevent expired locks being extended.

## Proof of Concept
https://github.com/code-423n4/2022-08-fiatdao/blob/main/contracts/VotingEscrow.sol#L493-L523

Call function function `increaseUnlockTime()` with an expired lock (locked[msg.sender].end < block.timestamp)
* Case 1: if sender's lock was not delegated to another address, function will be revert because of the requirement
https://github.com/code-423n4/2022-08-fiatdao/blob/main/contracts/VotingEscrow.sol#L511
* Case 2: if sender's lock was delegated to another address, function will not check anything and the lock can be extended.

But in case 1, sender’s lock was not delegated to another, the sender can delegate to new address with end time of lock equal to new end time. After that he can call `increaseUnlockTime()` and move to case 2. Then sender can undelegate and the lock will be extended, and sender will take back vote power.

Here is the script :
``` typescript=
describe("voting escrow", async () => {
    it("increase unlock time issue", async () => {
      await createSnapshot(provider);
      //alice creates lock
      let lockTime = WEEK + (await getTimestamp());
      await ve.connect(alice).createLock(lockAmount, lockTime);
      // bob creates lock
      lockTime = 50 * WEEK + (await getTimestamp());
      await ve.connect(bob).createLock(10 ** 8, lockTime);
      //pass 1 week, alice's lock is expired
      await ethers.provider.send("evm_mine", [await getTimestamp() + WEEK]);
      expect(await ve.balanceOf(alice.address)).to.eq(0);
      //alice can not increase unlock timme
      await expect(ve.connect(alice).increaseUnlockTime(lockTime)).to.be.revertedWith("Lock expired");
      //alice delegate to bob then can increase unlock time
      await ve.connect(alice).delegate(bob.address);
      await expect(ve.connect(alice).increaseUnlockTime(lockTime)).to.not.be.reverted;
      //alice delegate back herself
      await ve.connect(alice).delegate(alice.address);
      expect(await ve.balanceOf(alice.address)).to.gt(0);
    });
```

## Tools Used
Manual review
## Recommended Mitigation Steps
In every cases, expired locks should able to be extended -> should remove line https://github.com/code-423n4/2022-08-fiatdao/blob/main/contracts/VotingEscrow.sol#L511
