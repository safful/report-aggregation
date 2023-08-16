## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Basket becomes unusable if everybody burns their shares](https://github.com/code-423n4/2021-09-defiprotocol-findings/issues/64) 

# Handle

kenzo


# Vulnerability details

While handling the fees, the contract calculates the new ibRatio by dividing by totalSupply. This can be 0 leading to a division by 0.

## Impact
If everybody burns their shares, in the next mint, totalSupply will be 0, handleFees will revert, and so nobody will be able to use the basket anymore.

## Proof of Concept
Vulnerable line:
https://github.com/code-423n4/2021-09-defiProtocol/blob/52b74824c42acbcd64248f68c40128fe3a82caf6/contracts/contracts/Basket.sol#L124
You can add the following test to Basket.test.js and see that it reverts:
it("should divide by 0", async () => {
      await basket.connect(addr1).burn(await basket.balanceOf(addr1.address));
      await basket.connect(addr2).burn(await basket.balanceOf(addr2.address));

      await UNI.connect(addr1).approve(basket.address, ethers.BigNumber.from(1));
      await COMP.connect(addr1).approve(basket.address, ethers.BigNumber.from(1));
      await AAVE.connect(addr1).approve(basket.address, ethers.BigNumber.from(1));
      await basket.connect(addr1).mint(ethers.BigNumber.from(1));
  });


## Tools Used
Manual analysis, hardhat.

## Recommended Mitigation Steps
Add a check to handleFees: if totalSupply= 0, you can just return, no need to calculate new ibRatio / fees.
You might want to reset ibRatio to BASE at this point.

