## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [AccountantDelegate: The sweepInterest function sweeps an incorrect number of cnote.](https://github.com/code-423n4/2022-06-canto-v2-findings/issues/11) 

# Lines of code

https://github.com/Plex-Engineer/lending-market-v2/blob/ea5840de72eab58bec837bb51986ac73712fcfde/contracts/Accountant/AccountantDelegate.sol#L80-L99


# Vulnerability details

## Impact
In the sweepInterest function of the AccountantDelegate contract, the number of cnote sent to treasury should be cNoteToSweep instead of amtToSweep, as amtToSweep will normally be smaller than cNoteToSweep, which will cause the interest to be locked in the in the contract.
```
		uint amtToSweep = sub_(cNoteAmt, noteDiff); // amount to sweep in Note, 
		uint cNoteToSweep = div_(amtToSweep, exRate); // amount of cNote to sweep = amtToSweep(Note) / exRate

		cNoteToSweep = (cNoteToSweep > cNoteBal) ? cNoteBal :  cNoteToSweep; 
		bool success = cnote.transfer(treasury, amtToSweep);
		if (!success) {
			revert  SweepError(treasury , amtToSweep); //handles if transfer of tokens is not successful
		}

		TreasuryInterface Treas = TreasuryInterface(treasury);
		Treas.redeem(address(cnote),amtToSweep);
```
## Proof of Concept
https://github.com/Plex-Engineer/lending-market-v2/blob/ea5840de72eab58bec837bb51986ac73712fcfde/contracts/Accountant/AccountantDelegate.sol#L80-L99

## Tools Used
None
## Recommended Mitigation Steps
```diff
		uint amtToSweep = sub_(cNoteAmt, noteDiff); // amount to sweep in Note, 
		uint cNoteToSweep = div_(amtToSweep, exRate); // amount of cNote to sweep = amtToSweep(Note) / exRate

		cNoteToSweep = (cNoteToSweep > cNoteBal) ? cNoteBal :  cNoteToSweep; 
-		bool success = cnote.transfer(treasury, amtToSweep);
+               bool success = cnote.transfer(treasury, cNoteToSweep);
		if (!success) {
-			revert  SweepError(treasury , amtToSweep); //handles if transfer of tokens is not successful
+                       revert  SweepError(treasury , cNoteToSweep); //handles if transfer of tokens is not successful
		}

		TreasuryInterface Treas = TreasuryInterface(treasury);
-		Treas.redeem(address(cnote),amtToSweep);
+               Treas.redeem(address(cnote),cNoteToSweep);
```

