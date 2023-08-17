## Tags

- bug
- 3 (High Risk)
- primary issue
- satisfactory
- sponsor confirmed
- selected for report
- H-02

# [Non-existing revenue contract can be passed to claimRevenue to send all tokens to treasury](https://github.com/code-423n4/2022-11-debtdao-findings/issues/119) 

# Lines of code

https://github.com/debtdao/Line-of-Credit/blob/f32cb3eeb08663f2456bf6e2fba21e964da3e8ae/contracts/utils/SpigotLib.sol#L87


# Vulnerability details

## Impact
Neither `SpigotLib.claimRevenue` nor `SpigotLib._claimRevenue` check that the provided `revenueContract` was registered before. If this is not the case, `SpigotLib._claimRevenue` assumes that this is a revenue contract with push payments (because `self.settings[revenueContract].claimFunction` is 0) and just returns the difference since the last call to `claimRevenue`:
```solidity
       if(self.settings[revenueContract].claimFunction == bytes4(0)) {
            // push payments

            // claimed = total balance - already accounted for balance
            claimed = existingBalance - self.escrowed[token]; //@audit Rebasing tokens
            // underflow revert ensures we have more tokens than we started with and actually claimed revenue
        }
```
`SpigotLib.claimRevenue` will then read `self.settings[revenueContract].ownerSplit`, which is 0 for non-registered revenue contracts:
```solidity
uint256 escrowedAmount = claimed * self.settings[revenueContract].ownerSplit / 100;
```
Therefore, the whole `claimed` amount is sent to the treasury.

This becomes very problematic for revenue tokens that use push payments. An attacker (in practice the borrower) can just regularly call `claimRevenue` with this token and a non-existing revenue contract. All of the tokens that were sent to the spigot since the last call will be sent to the treasury and none to the escrow, i.e. a borrower can ensure that no revenue will be available for the lender, no matter what the configured split is.

## Proof Of Concept
As mentioned above, the attack pattern works for arbitrary tokens where one (or more) revenue contracts use push payments, i.e. where the balance of the Spigot increases from time to time. Then, the attacker just calls `claimRevenue` with a non-existing address. This is illustrated in the following diff:
```diff
--- a/contracts/tests/Spigot.t.sol
+++ b/contracts/tests/Spigot.t.sol
@@ -174,7 +174,7 @@ contract SpigotTest is Test {
         assertEq(token.balanceOf(address(spigot)), totalRevenue);
         
         bytes memory claimData;
-        spigot.claimRevenue(revenueContract, address(token), claimData);
+        spigot.claimRevenue(address(0), address(token), claimData);
```
Thanks to this small modification, all of the tokens are sent to the treasury and none are sent to the escrow.

## Recommended Mitigation Steps
Check that a revenue contract was registered before, revert if it does not.