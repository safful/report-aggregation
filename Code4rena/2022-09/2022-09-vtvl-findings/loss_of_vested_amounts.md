## Tags

- bug
- 3 (High Risk)
- sponsor confirmed
- edited-by-warden

# [Loss of vested amounts](https://github.com/code-423n4/2022-09-vtvl-findings/issues/475) 

# Lines of code

https://github.com/code-423n4/2022-09-vtvl/blob/f68b7f3e61dad0d873b5b5a1e8126b839afeab5f/contracts/VTVLVesting.sol#L418
https://github.com/code-423n4/2022-09-vtvl/blob/f68b7f3e61dad0d873b5b5a1e8126b839afeab5f/contracts/VTVLVesting.sol#L147-L151
https://github.com/code-423n4/2022-09-vtvl/blob/f68b7f3e61dad0d873b5b5a1e8126b839afeab5f/contracts/VTVLVesting.sol#L364


# Vulnerability details

## Impact

Vesting is a legal term that means the point in time where property is earned or gained by some person.
The VTVLVesting contract defines:
- a start time (Claim::startTimestamp) and an end time (Claim::endTimestamp) at which vesting starts and ends for a entitled user
- the calculated points in time when the fractions of the total amount are released and therefore can be withdrawn (which are defined by Claim::releaseIntervalSecs).
The entitled user can either withdraw after each interval elapses, or after the whole vesting period is over or any variant of the two options.

The administrator of the contract can revoke the claim for a user at any time, which for vesting assets is expected. For example an employee with a vesting stock allocation of 1000 shares vesting at each quarter over a period of 4 years, may resign after 2 years and therefore the only half of the shares would be vested and therefore sold by the employee. The employee can either sell them at each quarter, or before, or after resigning, in any case the half of the shares have vested and are by legal right owned by the employee.

The VTVLContract revoke has the following defects:
- it ignores the amount already vested and now yet withdrawn
- if called, say half-way the total period, just after claimer withdraws the already vested amount, it revokes only the right to vest the remaining part in future.
- if called, say half-way the total period, right before the claimer withdraws the already vested amount, it revokes both the already vested amount and the right to vest the remaining part in future.

Raising as high impact because it actually causes:
- loss of already vested amounts of a user with a valid claim that has already righteously vested a part but not withdrawn
- different outcomes depending on the order in which withdraw and revokeClaim functions are called which means that one of the two behavoiurs is certainly in conflict with the other causing a loss on one of the two sides, contract or claimer (by definition of Vesting rights, the claimer).
- lack of trust by the potential claimers/users whch can be at any time deprived of righteously vested amounts.

## Proof of Concept

The following two tests prove the behaviour difference when the order by which revokeClaim vs withdraw are called, whch shows that the vesting right is not guaranteed.

```solidity
  // NOTE: USES ORIGINAL REVOKE BEHAVIOUR
  it('sample revoke use case USER LOSE: employee withdraw immediately after resignation', async () => {
    const {tokenContract, vestingContract} = await createPrefundedVestingContract({tokenName, tokenSymbol, initialSupplyTokens});

    const startTimestamp = await getLastBlockTs() + 100;
    const endTimestamp = startTimestamp + 2000;
    const terminationTimestamp = startTimestamp + 1000 + 50; // half-way vesting, plus half release interval which shall be discarded
    const releaseIntervalSecs = 100;

    await vestingContract.createClaim(owner2.address, startTimestamp, endTimestamp, cliffReleaseTimestamp, releaseIntervalSecs, linearVestAmount, cliffAmount);

    // move clock to termination timestamp (half-way the vesting period plus a bit, but less than release interval seconds)
    await ethers.provider.send("evm_mine", [terminationTimestamp]);
    
    let availableAmt = await vestingContract.claimableAmount(owner2.address)
    // revoke the claim preserving the "already vested but not yet withdrawn amount"
    await (await vestingContract.revokeClaim(owner2.address)).wait();
    
    let userBalanceBefore = await tokenContract.balanceOf(owner2.address);
    await expect(vestingContract.connect(owner2).withdraw()).to.be.revertedWith('NO_ACTIVE_CLAIM');
    let userBalanceAfter = await tokenContract.balanceOf(owner2.address);

    // move the clock to the programmed end of vesting period
    await ethers.provider.send("evm_mine", [endTimestamp]);

    // cliffTimestamp < startTimestamp < terminationTimestamp, hence expected cliffAmount + (1/2 * anlinearVestAmount)
    let expectedVestedAmount = cliffAmount.add(linearVestAmount.div(2));

    // RESIGNING EMPLOYEE LOSES HIS VESTED AMOUNT BECAUSE OF WITHDRAWING IMMEDIATELY AFTER RESIGNATION
    expect(userBalanceAfter.sub(userBalanceBefore)).to.be.equal(0);
    // VTVLVesting CONTRACT TOOK ALREADY VESTED AMOUNT FROM OWNER2
    expect(await vestingContract.finalClaimableAmount(owner2.address)).to.be.equal(0);
  });

  // NOTE: USES ORIGINAL REVOKE BEHAVIOUR
  it('sample revoke use case USER WIN: employee withdraw immediately before resignation', async () => {
    const {tokenContract, vestingContract} = await createPrefundedVestingContract({tokenName, tokenSymbol, initialSupplyTokens});

    const startTimestamp = await getLastBlockTs() + 100;
    const endTimestamp = startTimestamp + 2000;
    const terminationTimestamp = startTimestamp + 1000 + 50; // half-way vesting, plus half release interval which shall be discarded
    const releaseIntervalSecs = 100;

    await vestingContract.createClaim(owner2.address, startTimestamp, endTimestamp, cliffReleaseTimestamp, releaseIntervalSecs, linearVestAmount, cliffAmount);

    // move clock to termination timestamp (half-way the vesting period plus a bit, but less than release interval seconds)
    await ethers.provider.send("evm_mine", [terminationTimestamp]);

    let userBalanceBefore = await tokenContract.balanceOf(owner2.address);
    await (await vestingContract.connect(owner2).withdraw()).wait();
    let userBalanceAfter = await tokenContract.balanceOf(owner2.address);

    // revoke the claim preserving the "already vested but not yet withdrawn amount"
    await (await vestingContract.revokeClaim(owner2.address)).wait();
    
    // move the clock to the programmed end of vesting period
    await ethers.provider.send("evm_mine", [endTimestamp]);

    console.log(userBalanceAfter.sub(userBalanceBefore));
    // RESIGNING EMPLOYEE RECEIVES HIS VESTED AMOUNT BY WITHDRAWING IMMEDIATELY BEFORE RESIGNATION
    expect(userBalanceAfter.sub(userBalanceBefore)).to.be.greaterThan(0);
    expect(await vestingContract.finalClaimableAmount(owner2.address)).to.be.equal(0);
  });
```solidity

## Tools Used

n/a

## Recommended Mitigation Steps

Below are, in order, a test and a diff/patch for a proposed fix. The proposed fix is just an idea at how to fix, or in other words, a way to preserve the already vested amount when claim is revoked.

The diff/patch add a deactivationTimestamp to claim, and a new revokeClaimProper that shall replace the revokeClaim function to correct the behaviour.
The deactivationTimestamp is used to track the deactivation time for the claim in order to preserve the amount vested so far and allow the user to withdraw the amount righteously earned so far. The _baseVestedAmount and hasActiveClaim have been updated to do proper math when isActive is false but deactivationTimestamp is greater than 0.

The finalVestedAmount has been update to show the "what would be" amount if the vesting would have reached the claim endTimestamp while the finalClaimableAmount takes into consideration the deactivationTimestamp if the claim has been revoked.

The test shows that the already vested amount (cliff + half way linear vesting) is preserved.

```solidity
diff --git a/contracts/VTVLVesting.sol b/contracts/VTVLVesting.sol
index 133f19f..7ab955c 100644
--- a/contracts/VTVLVesting.sol
+++ b/contracts/VTVLVesting.sol
@@ -34,6 +34,7 @@ contract VTVLVesting is Context, AccessProtected {
         // Gives us a range from 1 Jan 1970 (Unix epoch) up to approximately 35 thousand years from then (2^40 / (365 * 24 * 60 * 60) ~= 35k)
         uint40 startTimestamp; // When does the vesting start (40 bits is enough for TS)
         uint40 endTimestamp; // When does the vesting end - the vesting goes linearly between the start and end timestamps
+        uint40 deactivationTimestamp;
         uint40 cliffReleaseTimestamp; // At which timestamp is the cliffAmount released. This must be <= startTimestamp
         uint40 releaseIntervalSecs; // Every how many seconds does the vested amount increase. 
         
@@ -108,7 +109,7 @@ contract VTVLVesting is Context, AccessProtected {
 
         // We however still need the active check, since (due to the name of the function)
         // we want to only allow active claims
-        require(_claim.isActive == true, "NO_ACTIVE_CLAIM");
+        require(_claim.isActive == true || _claim.deactivationTimestamp > 0, "NO_ACTIVE_CLAIM");
 
         // Save gas, omit further checks
         // require(_claim.linearVestAmount + _claim.cliffAmount > 0, "INVALID_VESTED_AMOUNT");
@@ -144,20 +145,20 @@ contract VTVLVesting is Context, AccessProtected {
     @param _claim The claim in question
     @param _referenceTs Timestamp for which we're calculating
      */
-    function _baseVestedAmount(Claim memory _claim, uint40 _referenceTs) internal pure returns (uint112) {
+    function _baseVestedAmount(Claim memory _claim, uint40 _referenceTs, uint40 vestEndTimestamp) internal pure returns (uint112) {
         uint112 vestAmt = 0;
-        
-        // the condition to have anything vested is to be active
-        if(_claim.isActive) {
+            
+        if(_claim.isActive || _claim.deactivationTimestamp > 0) {
             // no point of looking past the endTimestamp as nothing should vest afterwards
             // So if we're past the end, just get the ref frame back to the end
-            if(_referenceTs > _claim.endTimestamp) {
-                _referenceTs = _claim.endTimestamp;
+            if(_referenceTs > vestEndTimestamp) {
+                _referenceTs = vestEndTimestamp;
             }
 
             // If we're past the cliffReleaseTimestamp, we release the cliffAmount
             // We don't check here that cliffReleaseTimestamp is after the startTimestamp 
-            if(_referenceTs >= _claim.cliffReleaseTimestamp) { // @audit is _claim.require(cliffReleaseTimestamp < _claim.endTimestamp) ?
+            if(_referenceTs >= _claim.cliffReleaseTimestamp) {  // @audit note  cliffReleaseTimestamp cannot? be zero without cliffamoutn being zero
+                // @audit NOTE: (cliffReleaseTimestamp is always <= _startTimestamp <= endTimestamp, or 0 if no vesting)
                 vestAmt += _claim.cliffAmount;
             }
 
@@ -195,7 +196,8 @@ contract VTVLVesting is Context, AccessProtected {
     */
     function vestedAmount(address _recipient, uint40 _referenceTs) public view returns (uint112) {
         Claim storage _claim = claims[_recipient];
-        return _baseVestedAmount(_claim, _referenceTs);
+        uint40 vestEndTimestamp = _claim.isActive ? _claim.endTimestamp : _claim.deactivationTimestamp;
+        return _baseVestedAmount(_claim, _referenceTs, vestEndTimestamp);
     }
 
     /**
@@ -205,7 +207,18 @@ contract VTVLVesting is Context, AccessProtected {
      */
     function finalVestedAmount(address _recipient) public view returns (uint112) {
         Claim storage _claim = claims[_recipient];
-        return _baseVestedAmount(_claim, _claim.endTimestamp);
+        return _baseVestedAmount(_claim, _claim.endTimestamp, _claim.endTimestamp);
+    }
+
+    /**
+    @notice Calculates how much wil be possible to claim at the end of vesting date, by subtracting the already withdrawn
+            amount from the vestedAmount at this moment. Vesting date is either the end timestamp or the deactivation timestamp.
+    @param _recipient - The address for whom we're calculating
+    */
+    function finalClaimableAmount(address _recipient) external view returns (uint112) {
+        Claim storage _claim = claims[_recipient];
+        uint40 vestEndTimestamp = _claim.isActive ? _claim.endTimestamp : _claim.deactivationTimestamp;
+        return _baseVestedAmount(_claim, vestEndTimestamp, vestEndTimestamp) - _claim.amountWithdrawn;
     }
     
     /**
@@ -214,7 +227,8 @@ contract VTVLVesting is Context, AccessProtected {
     */
     function claimableAmount(address _recipient) external view returns (uint112) {
         Claim storage _claim = claims[_recipient];
-        return _baseVestedAmount(_claim, uint40(block.timestamp)) - _claim.amountWithdrawn;
+        uint40 vestEndTimestamp = _claim.isActive ? _claim.endTimestamp : _claim.deactivationTimestamp;
+        return _baseVestedAmount(_claim, uint40(block.timestamp), vestEndTimestamp) - _claim.amountWithdrawn;
     }
     
     /** 
@@ -280,6 +294,7 @@ contract VTVLVesting is Context, AccessProtected {
         Claim memory _claim = Claim({
             startTimestamp: _startTimestamp,
             endTimestamp: _endTimestamp,
+            deactivationTimestamp: 0,
             cliffReleaseTimestamp: _cliffReleaseTimestamp,
             releaseIntervalSecs: _releaseIntervalSecs,
             cliffAmount: _cliffAmount,
@@ -436,6 +451,30 @@ contract VTVLVesting is Context, AccessProtected {
         emit ClaimRevoked(_recipient, amountRemaining, uint40(block.timestamp), _claim);
     }
 
+    function revokeClaimProper(address _recipient) external onlyAdmin hasActiveClaim(_recipient) {
+        // Fetch the claim
+        Claim storage _claim = claims[_recipient];
+        // Calculate what the claim should finally vest to
+        uint112 finalVestAmt = finalVestedAmount(_recipient);
+
+        // No point in revoking something that has been fully consumed
+        // so require that there be unconsumed amount
+        require( _claim.amountWithdrawn < finalVestAmt, "NO_UNVESTED_AMOUNT");
+
+        _claim.isActive = false;
+        _claim.deactivationTimestamp = uint40(block.timestamp);
+
+        uint112 vestedSoFarAmt = vestedAmount(_recipient, uint40(block.timestamp));
+        // The amount that is "reclaimed" is equal to the total allocation less what was already
+        // vested without the part that was already withdrawn.
+        uint112 amountRemaining = finalVestAmt - (vestedSoFarAmt - _claim.amountWithdrawn);
+
+        numTokensReservedForVesting -= amountRemaining; // Reduces the allocation
+
+        // Tell everyone a claim has been revoked.
+        emit ClaimRevoked(_recipient, amountRemaining, uint40(block.timestamp), _claim);
+    }
+
     /**
     @notice Withdraw a token which isn't controlled by the vesting contract.
     @dev This contract controls/vests token at "tokenAddress". However, someone might send a different token. 

```