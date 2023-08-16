## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Public functions to external](https://github.com/code-423n4/2021-11-malt-findings/issues/163) 

# Handle

robee


# Vulnerability details

The following functions could be set external to save gas and improve code quality. 
External call cost is less expensive than of public functions. 

        The function getRewardOwnershipFraction in AbstractRewardMine.sol could be set external
        The function balanceOfStakePadding in AbstractRewardMine.sol could be set external
        The function withdrawnBalance in AbstractRewardMine.sol could be set external
        The function earned in AbstractRewardMine.sol could be set external
        The function setMiningService in AbstractRewardMine.sol could be set external
        The function totalReleasedReward in AbstractRewardMine.sol could be set external
        The function totalStakePadding in AbstractRewardMine.sol could be set external
        The function totalBonded in AbstractRewardMine.sol could be set external
        The function totalWithdrawn in AbstractRewardMine.sol could be set external
        The function setRewardToken in AbstractRewardMine.sol could be set external
        The function onBond in AbstractRewardMine.sol could be set external
        The function withdrawAll in AbstractRewardMine.sol could be set external
        The function onUnbond in AbstractRewardMine.sol could be set external
        The function balanceOfBonded in AbstractRewardMine.sol could be set external
        The function verifyTransfer in AbstractTransferVerification.sol could be set external
        The function getAuctionCore in Auction.sol could be set external
        The function getAuctionPrices in Auction.sol could be set external
        The function getAuction in Auction.sol could be set external
        The function auctionActive in Auction.sol could be set external
        The function getAuctionCommitments in Auction.sol could be set external
        The function isAuctionFinalized in Auction.sol could be set external
        The function balanceOfArbTokens in Auction.sol could be set external
        The function auctionExists in Auction.sol could be set external
        The function consult in AuctionBurnReserveSkew.sol could be set external
        The function getAverageParticipation in AuctionBurnReserveSkew.sol could be set external
        The function addAbovePegObservation in AuctionBurnReserveSkew.sol could be set external
        The function getPegDeltaFrequency in AuctionBurnReserveSkew.sol could be set external
        The function addBelowPegObservation in AuctionBurnReserveSkew.sol could be set external
        The function getRealBurnBudget in AuctionBurnReserveSkew.sol could be set external
        The function earlyExitReturn in AuctionEscapeHatch.sol could be set external
        The function setReplenishingIndex in AuctionParticipant.sol could be set external
        The function usableBalance in AuctionParticipant.sol could be set external
        The function outstandingArbTokens in AuctionParticipant.sol could be set external
        The function getAllAuctionIds in AuctionParticipant.sol could be set external
        The function setupParticipant in AuctionParticipant.sol could be set external
        The function usableBalance in AuctionPool.sol could be set external
        The function setBonding in AuctionPool.sol could be set external
        The function totalReleasedReward in AuctionPool.sol could be set external
        The function totalBonded in AuctionPool.sol could be set external
        The function setForfeitDestination in AuctionPool.sol could be set external
        The function onUnbond in AuctionPool.sol could be set external
        The function totalDeclaredReward in AuctionPool.sol could be set external
        The function balanceOfBonded in AuctionPool.sol could be set external
        The function setMiningService in Bonding.sol could be set external
        The function epochData in Bonding.sol could be set external
        The function totalBonded in Bonding.sol could be set external
        The function bondedEpoch in Bonding.sol could be set external
        The function setDAO in Bonding.sol could be set external
        The function setDexHandler in Bonding.sol could be set external
        The function setCurrentEpoch in Bonding.sol could be set external
        The function bondToAccount in Bonding.sol could be set external
        The function averageBondedValue in Bonding.sol could be set external
        The function balanceOfBonded in Bonding.sol could be set external
        The function deploy in Create2Deployer.sol could be set external
        The function getEpochStartTime in DAO.sol could be set external
        The function epochsPerYear in DAO.sol could be set external
        The function mint in DAO.sol could be set external
        The function setEpochLength in DAO.sol could be set external
        The function setMaltToken in DAO.sol could be set external
        The function maltMarketPrice in UniswapHandler.sol could be set external
        The function reserves in UniswapHandler.sol could be set external
        The function constructor in ERC20Permit.sol could be set external
        The function setBonding in ERC20VestedMine.sol could be set external
        The function totalReleasedReward in ERC20VestedMine.sol could be set external
        The function totalBonded in ERC20VestedMine.sol could be set external
        The function setDistributor in ERC20VestedMine.sol could be set external
        The function onUnbond in ERC20VestedMine.sol could be set external
        The function totalDeclaredReward in ERC20VestedMine.sol could be set external
        The function balanceOfBonded in ERC20VestedMine.sol could be set external
        The function handleForfeit in ForfeitHandler.sol could be set external
        The function totalUsefulCollateral in ImpliedCollateralService.sol could be set external
        The function getCollateralValueInMalt in ImpliedCollateralService.sol could be set external
        The function collateralDeficit in LiquidityExtension.sol could be set external
        The function reserveRatio in LiquidityExtension.sol could be set external
        The function hasMinimumReserves in LiquidityExtension.sol could be set external
        The function constructor in Malt.sol could be set external
        The function burn in Malt.sol could be set external
        The function mint in Malt.sol could be set external
        The function smoothedMaltPrice in MaltDataLab.sol could be set external
        The function trackReserveRatio in MaltDataLab.sol could be set external
        The function maltInPoolAverage in MaltDataLab.sol could be set external
        The function smoothedMaltInPool in MaltDataLab.sol could be set external
        The function reserveRatioAverage in MaltDataLab.sol could be set external
        The function maltPriceAverage in MaltDataLab.sol could be set external
        The function smoothedReserves in MaltDataLab.sol could be set external
        The function smoothedReserveRatio in MaltDataLab.sol could be set external
        The function balanceOfRewards in MiningService.sol could be set external
        The function removeRewardMine in MiningService.sol could be set external
        The function earned in MiningService.sol could be set external
        The function withdrawRewardsForAccount in MiningService.sol could be set external
        The function setReinvestor in MiningService.sol could be set external
        The function withdrawAccountRewards in MiningService.sol could be set external
        The function setBonding in MiningService.sol could be set external
        The function isMineActive in MiningService.sol could be set external
        The function onBond in MiningService.sol could be set external
        The function numberOfMines in MiningService.sol could be set external
        The function onUnbond in MiningService.sol could be set external
        The function addRewardMine in MiningService.sol could be set external
        The function getValue in MovingAverage.sol could be set external
        The function getValueWithLookback in MovingAverage.sol could be set external
        The function isWhitelisted in PoolTransferVerification.sol could be set external
        The function setPool in PoolTransferVerification.sol could be set external
        The function addToWhitelist in PoolTransferVerification.sol could be set external
        The function verifyTransfer in PoolTransferVerification.sol could be set external
        The function setThreshold in PoolTransferVerification.sol could be set external
        The function setPriceLookback in PoolTransferVerification.sol could be set external
        The function removeFromWhitelist in PoolTransferVerification.sol could be set external
        The function setForfeitor in RewardDistributor.sol could be set external
        The function addFocalLengthUpdater in RewardDistributor.sol could be set external
        The function setRewardMine in RewardDistributor.sol could be set external
        The function setBonding in RewardDistributor.sol could be set external
        The function forfeit in RewardDistributor.sol could be set external
        The function vest in RewardDistributor.sol could be set external
        The function setThrottler in RewardDistributor.sol could be set external
        The function setRewardToken in RewardDistributor.sol could be set external
        The function removeFocalLengthUpdater in RewardDistributor.sol could be set external
        The function decrementRewards in RewardDistributor.sol could be set external
        The function totalDeclaredReward in RewardDistributor.sol could be set external
        The function setFocalLength in RewardDistributor.sol could be set external
        The function averageAPR in RewardThrottle.sol could be set external
        The function targetEpochProfit in RewardThrottle.sol could be set external
        The function epochData in RewardThrottle.sol could be set external
        The function targetAPR in RewardThrottle.sol could be set external
        The function getTargets in RewardThrottle.sol could be set external
        The function handleReward in RewardThrottle.sol could be set external
        The function checkRewardUnderflow in RewardThrottle.sol could be set external
        The function epochAPR in RewardThrottle.sol could be set external
        The function costBasis in SwingTrader.sol could be set external
        The function setLpProfitCut in SwingTrader.sol could be set external
        The function addVerifier in TransferService.sol could be set external
        The function numberOfVerifiers in TransferService.sol could be set external
        The function verifyTransfer in TransferService.sol could be set external
        The function removeVerifier in TransferService.sol could be set external


