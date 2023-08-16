## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Unused declared local variables](https://github.com/code-423n4/2021-11-malt-findings/issues/158) 

# Handle

robee


# Vulnerability details

Unused local variables are gas consuming, since the initial value assignment costs gas. And are 
a bad code practice. Removing those variables will decrease the gas cost and improve code quality. 
This is a full list of all the unused storage variables we found in your code base. 
The format is <solidity file>, <in which function we found it>, <unused local variable name>: 

        AbstractRewardMine.sol, _handleStakePadding, totalRewardedWithStakePadding
        AbstractRewardMine.sol, _handleStakePadding, INITIAL_STAKE_SHARE_MULTIPLE
        AbstractRewardMine.sol, _handleStakePadding, bondedTotal
        Auction.sol, _finalizeAuction, avgMaltPrice
        AuctionParticipant.sol, claim, replenishingId
        AuctionParticipant.sol, claim, claimableTokens
        AuctionParticipant.sol, claim, claimable
        Create2Deployer.sol, deploy, addr
        UniswapHandler.sol, removeBuyer, buyer
        MaltDataLab.sol, trackPoolReserves, rewardDecimals
        MovingAverage.sol, update, elapsedSamples
        MovingAverage.sol, updateCumulative, elapsedSamples
        StabilizerNode.sol, stabilize, exchangeRate
        StabilizerNode.sol, _startAuction, decimals
        SwingTrader.sol, sellMalt, maltDecimals
        SwingTrader.sol, costBasis, maltDecimals
        TransferService.sol, removeVerifier, verifier


