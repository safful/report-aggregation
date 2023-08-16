## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Storage double reading. Could save SLOAD](https://github.com/code-423n4/2021-11-malt-findings/issues/161) 

# Handle

robee


# Vulnerability details

Reading a storage variable is gas costly (SLOAD). In cases of multiple read of a storage variable in the same scope, caching the first read (i.e saving as a local variable) can save gas and decrease the
 overall gas uses. The following is a list of functions and the storage variables that you read twice: 

        AbstractRewardMine.sol Variable miningService is read 3 times in the function:  setMiningService
        Auction.sol Variable currentAuctionId is read 3 times in the function:  purchaseArbitrageTokens
        Auction.sol Variable nextCommitmentId is read 2 times in the function:  purchaseArbitrageTokens
        Auction.sol Variable currentAuctionId is read 4 times in the function:  _checkAuctionFinalization
        Auction.sol Variable replenishingAuctionId is read 4 times in the function:  allocateArbRewards
        Auction.sol Variable stabilizerNode is read 2 times in the function:  setStabilizerNode
        Auction.sol Variable amender is read 2 times in the function:  setAuctionAmender
        AuctionBurnReserveSkew.sol Variable auctionAverageLookback is read 2 times in the function:  getPegDeltaFrequency
        AuctionBurnReserveSkew.sol Variable count is read 2 times in the function:  addAbovePegObservation
        AuctionBurnReserveSkew.sol Variable count is read 2 times in the function:  addBelowPegObservation
        AuctionBurnReserveSkew.sol Variable stabilizerNode is read 2 times in the function:  setNewStabilizerNode
        AuctionParticipant.sol Variable replenishingIndex is read 2 times in the function:  claim
        AuctionParticipant.sol Variable replenishingIndex is read 2 times in the function:  setReplenishingIndex
        AuctionPool.sol Variable forfeitedRewards is read 2 times in the function:  _checkForForfeit
        AuctionPool.sol Variable forfeitedRewards is read 3 times in the function:  _handleRewardDistribution
        UniswapHandler.sol Variable router is read 2 times in the function:  addLiquidity
        MaltDataLab.sol Variable UPDATER_ROLE is read 2 times in the function:  initialize
        MiningService.sol Variable reinvestor is read 2 times in the function:  setReinvestor
        MiningService.sol Variable bonding is read 2 times in the function:  setBonding
        MovingAverage.sol Variable UPDATER_ROLE is read 2 times in the function:  initialize
        MovingAverage.sol Variable cumulativeValue is read 11 times in the function:  update
        MovingAverage.sol Variable blockTimestampLast is read 5 times in the function:  update
        MovingAverage.sol Variable activeSamples is read 2 times in the function:  update
        MovingAverage.sol Variable sampleLength is read 3 times in the function:  update
        MovingAverage.sol Variable cumulativeValue is read 10 times in the function:  updateCumulative
        MovingAverage.sol Variable blockTimestampLast is read 4 times in the function:  updateCumulative
        MovingAverage.sol Variable activeSamples is read 2 times in the function:  updateCumulative
        MovingAverage.sol Variable sampleLength is read 2 times in the function:  updateCumulative
        MovingAverage.sol Variable activeSamples is read 2 times in the function:  _createNewSample
        MovingAverage.sol Variable counter is read 2 times in the function:  setSampleMemory
        RewardReinvestor.sol Variable dexHandler is read 2 times in the function:  _bondAccount
        RewardReinvestor.sol Variable treasury is read 2 times in the function:  _bondAccount
        RewardDistributor.sol Variable FOCAL_LENGTH_UPDATER_ROLE is read 2 times in the function:  initialize
        RewardDistributor.sol Variable focalLength is read 2 times in the function:  _resetFocalPoint
        RewardDistributor.sol Variable focalID is read 3 times in the function:  _incrementFocalPoint
        RewardDistributor.sol Variable throttler is read 2 times in the function:  setThrottler
        RewardDistributor.sol Variable rewardMine is read 2 times in the function:  setRewardMine
        RewardOverflowPool.sol Variable throttler is read 2 times in the function:  setThrottler
        RewardThrottle.sol Variable _activeEpoch is read 3 times in the function:  handleReward
        StabilizerNode.sol Variable stabilizeWindowEnd is read 2 times in the function:  stabilize
        StabilizerNode.sol Variable lastStabilize is read 2 times in the function:  stabilize
        StabilizerNode.sol Variable liquidityExtension is read 2 times in the function:  _replenishLiquidityExtension
        StabilizerNode.sol Variable auction is read 2 times in the function:  setAuctionContract
        SwingTrader.sol Variable deployedCapital is read 2 times in the function:  buyMalt
        SwingTrader.sol Variable deployedCapital is read 2 times in the function:  sellMalt


