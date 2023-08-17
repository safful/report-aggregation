## Tags

- bug
- 2 (Med Risk)
- disagree with severity
- sponsor confirmed

# [THE FIRST AMM STAKER MAY NOT RECEIVE ACCORDING REWARDS BECAUSE OF POOR CHECKPOINTS](https://github.com/code-423n4/2022-05-backd-findings/issues/111) 

# Lines of code

https://github.com/code-423n4/2022-05-backd/blob/2a5664d35cde5b036074edef3c1369b984d10010/protocol/contracts/tokenomics/AmmGauge.sol#L56
https://github.com/code-423n4/2022-05-backd/blob/2a5664d35cde5b036074edef3c1369b984d10010/protocol/contracts/tokenomics/AmmGauge.sol#L140


# Vulnerability details

## Impact
The first staker within the `AmmGauge` may not get the rewards if the pool is not checkpointed right after he stakes and before he wants to claim the rewards.

## Proof of Concept
A testing environment that reproduces how the protocol is going to be deployed and managed is used to evaluate this case under the following assumptions and simplifications.


1) The inflation rate is fixed for simplicity (`0.001`). 
2) For the testing environment performed by the team, a DummyERC20 was used as testing token. The same is done on the exploit environment.
3) The minting of tokens impact both on the inflation calculation and their balance. But this test evaluates the states just before minting (claimable balances). Following how the pools are updated, they are checkpointed in the end of the `_executeInflationRateUpdate` call. Not while staking.

In order to illustrate this scenario we will show both the vulnerable and non vulnerable situations.

Vulnerable Situation:
1) Alice, Bob, Charlie and David are future users of the pool. They all notice the inception of this project and decide to stake.
2) They all stake the same amount. Their transactions are mined with 1min of difference starting from Alice and finishing with David.
3) There is no external pool checkpoint between Alice and Bob (besides the one that is triggered when Bob stakes).
4) Sometime happens and they all want to check their accumulated reward balance. Alice accumulated much less than the others.

Non Vulnerable Situation:
- The same as before but calling externally `_poolCheckpoint()` between Alice stake call and Bobs' and before checking the accumulated rewards.

The code to show this has a `secureCheckpoints` toggle that can be set as true or false to trigger (or not) the intermediate poolCheckpoints.

        it('First Staker Rewards Calculation', async function () { 
            
            let secureCheckpoints = false;
            let currentShare, currentStakedIntegral, balances;
            await this.ammgauge.poolCheckpoint();
            await ethers.provider.send("evm_increaseTime", [1 * 24 * 60 * 60]); // 10 days
            
            const updateStates = async (from) => { 
                currentShare = await this.ammgauge.perUserShare(from.address);
                currentStakedIntegral = await this.ammgauge.perUserStakedIntegral(from.address);
                balances = await this.ammgauge.balances(from.address);
            }

            const stake = async (to, amount) => {
                await updateStates(to)
                console.log(" ")
                // Balance before
                let balanceBefore = await this.ammgauge.balances(to.address);
                // Stake
                await this.ammgauge.connect(to).stake(amount);
                expect(await this.ammgauge.balances(to.address)).to.be.eq(balanceBefore.add(amount));
                await updateStates(to);
                console.log(" ")
            }

            const unstake = async (to, amount) => {
                await updateStates(to)
                console.log(" ")
                // Balance before
                let balanceBefore = await this.ammgauge.balances(to.address);
                // Stake
                await this.ammgauge.connect(to).unstake(amount);
                expect(await this.ammgauge.balances(to.address)).to.be.eq(balanceBefore.sub(amount));
                await updateStates(to);
                console.log(" ")
            }

            // Each user stakes tokens
            let initialStaking = ethers.utils.parseEther("10")
            console.log(" ")
            console.log("USERS STAKE");
            for (const user of users) {
            await stake(user, initialStaking)
            if(secureCheckpoints){await this.ammgauge.poolCheckpoint()};
            await ethers.provider.send("evm_increaseTime", [60 * 60]); // 1hr between stakes
            }
            console.log(" ")

            await ethers.provider.send("evm_increaseTime", [ 5 * 24 * 60 * 60]); // 5 days
            if(secureCheckpoints){await this.ammgauge.poolCheckpoint()};

            let claimableRewards = [];
            let claimedRewards = [];
            console.log(" ")
            console.log("USERS CLAIMABLE REWARDS AFTER 5 days");
            console.log(" ")
            for (const user of users) {
                let stepClaimable = await this.ammgauge.claimableRewards(user.address);
                claimableRewards.push(ethers.utils.formatEther(stepClaimable))

                let rewardsClaim = await (await this.ammgauge.claimRewards(user.address)).wait()
                claimedRewards.push(ethers.utils.formatEther(rewardsClaim.logs[0]["data"]))
            }

            console.log("Claimable calculated")
            console.log("   ALICE - BOB -  CHARLIE - DAVID")
            console.log(claimableRewards)

            console.log(" ")
            console.log("Effectively Claimed")
            console.log("   ALICE - BOB -  CHARLIE - DAVID")
            console.log(claimableRewards)
        })

The outputs for both cases are shown on the following chart. The initial staking amount is 10eth amount of the DummyERC20 token.

|         | Without Checkpoints | With Checkpoints |
|:-------:|:-------------------:|:----------------:|
|  Alice  |         6.6         |       115.5      |
|   Bob   |        111.9        |       111.9      |
| Charlie |        110.1        |       110.1      |
|  David  |        108.9        |       108.9      |


## Recommended Mitigation Steps
- Check how is calculated the staking variables while the pool has no tokens staked and also how the updates and checkpoints are performed.

