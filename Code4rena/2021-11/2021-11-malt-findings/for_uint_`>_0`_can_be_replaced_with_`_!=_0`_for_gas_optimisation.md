## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [For uint `> 0` can be replaced with ` != 0` for gas optimisation](https://github.com/code-423n4/2021-11-malt-findings/issues/271) 

# Handle

0x0x0x


# Vulnerability details

## Impact

`!= 0` is a cheaper operation compared to `> 0`, when dealing with `uint`.

## Occurences

```

./AbstractRewardMine.sol:147:    if (rewardDenominator > 0) {
./Auction.sol:219:    require(amountTokens > 0, "No claimable Arb tokens");
./Auction.sol:265:    return auction.endingTime > 0 && (now >= auction.endingTime || auction.finalPrice > 0 || auction.commitments >= auction.maxCommitments);
./Auction.sol:385:    return auction.startingTime > 0;
./Auction.sol:639:    if (realBurnBudget > 0) {
./Auction.sol:659:    require(auction.startingTime > 0, "No auction available for the given id");
./Auction.sol:663:    if (auction.maltPurchased > 0) {
./Auction.sol:861:          if (auction.commitments > 0 || !auction.finalized) {
./Auction.sol:894:    require(_length > 0, "Length must be larger than 0");
./Auction.sol:972:    require(_split > 0 && _split <= 10000, "Must be between 0-100%");
./Auction.sol:980:    require(_maxEnd > 0 && _maxEnd <= 1000, "Must be between 0-100%");
./Auction.sol:988:    require(_lookback > 0, "Must be above 0");
./Auction.sol:996:    require(_lookback > 0, "Must be above 0");
./Auction.sol:1004:    require(_bps > 0 && _bps < 1000, "Must be between 0-100%");
./Auction.sol:1012:    require(_threshold > 0, "Must be between greater than 0");
./AuctionBurnReserveSkew.sol:109:    if (aggregate.maxCommitments > 0) {
./AuctionBurnReserveSkew.sol:190:    require(_lookback > 0, "Cannot have zero lookback period");
./AuctionEscapeHatch.sol:191:    require(amount > 0, "Nothing to claim");
./AuctionEscapeHatch.sol:222:    require(_earlyExitBps > 0 && _earlyExitBps <= 1000, "Must be between 0-100%");
./AuctionEscapeHatch.sol:230:    require(_period > 0, "Cannot have 0 lookback period");
./AuctionPool.sol:118:    if (globalRewarded > 0 && userReward > 0) {
./AuctionPool.sol:125:    if (forfeitAmount > 0) {
./AuctionPool.sol:129:    if (declaredRewardDecrease > 0) {
./AuctionPool.sol:141:    if (forfeitedRewards > 0) {
./Bonding.sol:87:    require(amount > 0, "Cannot bond 0");
./Bonding.sol:97:    require(amount > 0, "Cannot unbond 0");
./Bonding.sol:101:    require(bondedBalance > 0, "< bonded balance");
./Bonding.sol:117:    require(amount > 0, "Cannot unbond 0");
./Bonding.sol:121:    require(bondedBalance > 0, "< bonded balance");
./Bonding.sol:283:      if (diff > 0) {
./DAO.sol:47:    if (offeringMint > 0) {
./DAO.sol:78:    require(amount > 0, "Cannot have zero amount");
./DAO.sol:94:    require(_length > 0, "Cannot have zero length epochs");
./ERC20VestedMine.sol:93:    if (globalRewarded > 0 && userReward > 0) {
./ERC20VestedMine.sol:115:    if (forfeitReward > 0) {
./ERC20VestedMine.sol:119:    if (declaredRewardDecrease > 0) {
./ForfeitHandler.sol:49:    if (swingTraderCut > 0) {
./ForfeitHandler.sol:53:    if (treasuryCut > 0) {
./ImpliedCollateralService.sol:64:    if (maxAmount > 0) {
./ImpliedCollateralService.sol:68:    if (maxAmount > 0) {
./ImpliedCollateralService.sol:71:      // if (maxAmount > 0) {
./ImpliedCollateralService.sol:74:      //   if (maxAmount > 0) {
./LiquidityExtension.sol:161:    require(_ratio > 0 && _ratio <= 100, "Must be between 0 and 100");
./MaltDataLab.sol:234:    require(_price > 0, "Cannot have 0 price");
./MaltDataLab.sol:242:    require(_lookback > 0, "Cannot have 0 lookback");
./MaltDataLab.sol:250:    require(_lookback > 0, "Cannot have 0 lookback");
./MaltDataLab.sol:258:    require(_lookback > 0, "Cannot have 0 lookback");
./MovingAverage.sol:385:    if (oldSample.timestamp > 0 && activeSamples > 1) {
./MovingAverage.sol:412:    require(_sampleLength > 0, "Cannot have 0 second sample length");
./MovingAverage.sol:428:    require(_sampleMemory > 0, "Cannot have sample memroy of 0");
./PoolTransferVerification.sol:76:    require(newThreshold > 0 && newThreshold < 10000, "Threshold must be between 0-100%");
./PoolTransferVerification.sol:85:    require(lookback > 0, "Cannot have 0 lookback");
./RewardReinvestor.sol:93:    require(rewardLiquidity > 0, "Cannot reinvest 0");
./RewardReinvestor.sol:115:    if (maltBalance > 0) {
./RewardReinvestor.sol:119:    if (rewardTokenBalance > 0) {
./RewardSystem/RewardDistributor.sol:144:    require(reward > 0, "Cannot declare 0 reward");
./RewardSystem/RewardDistributor.sol:266:    if (amount > 0) {
./RewardSystem/RewardDistributor.sol:277:    if (amount > 0) {
./RewardSystem/RewardOverflowPool.sol:76:    require(_maxFulfillment > 0, "Can't have 0 max fulfillment");
./RewardSystem/RewardThrottle.sol:85:    if (aprTarget > 0 && _epochAprGivenReward(epoch, balance) > aprTarget) {
./RewardSystem/RewardThrottle.sol:89:      if (remainder > 0) {
./RewardSystem/RewardThrottle.sol:271:          if (underflow > 0) {
./RewardSystem/RewardThrottle.sol:323:    require(_smoothingPeriod > 0, "No zero smoothing period");
./StabilizerNode.sol:270:    if (callerCut > 0) {
./StabilizerNode.sol:274:    if (auctionPoolCut > 0) {
./StabilizerNode.sol:278:    if (swingTraderCut > 0) {
./StabilizerNode.sol:282:    if (treasuryCut > 0) {
./StabilizerNode.sol:286:    if (daoCut > 0) {
./StabilizerNode.sol:290:    if (lpCut > 0) {
./StabilizerNode.sol:359:    require(_period > 0, "Must be greater than 0");
./StabilizerNode.sol:406:    require(_incentive > 0, "No negative incentive");
./StabilizerNode.sol:417:    require(amount > 0, "No negative damping");
./StabilizerNode.sol:449:    require(_upper > 0 && _lower > 0, "Must be above 0");
./StabilizerNode.sol:488:    require(_maxContribution > 0 && _maxContribution <= 100, "Must be between 0 and 100");
./StabilizerNode.sol:552:    require(_period > 0, "Cannot have 0 period");
./StabilizerNode.sol:561:    require(_distance > 0 && _distance < 1000, "Override must be between 0-100%");
./StabilizerNode.sol:570:    require(_period > 0, "Cannot have 0 period");
./SwingTrader.sol:136:    if (profit > 0) {
./libraries/SafeBurnMintableERC20.sol:70:        if (returndata.length > 0) { // Return data is optional
./libraries/UniswapV2Library.sol:37:        require(amountA > 0, 'UniswapV2Library: INSUFFICIENT_AMOUNT');
./libraries/UniswapV2Library.sol:38:        require(reserveA > 0 && reserveB > 0, 'UniswapV2Library: INSUFFICIENT_LIQUIDITY');
./libraries/UniswapV2Library.sol:44:        require(amountIn > 0, 'UniswapV2Library: INSUFFICIENT_INPUT_AMOUNT');
./libraries/UniswapV2Library.sol:45:        require(reserveIn > 0 && reserveOut > 0, 'UniswapV2Library: INSUFFICIENT_LIQUIDITY');
./libraries/UniswapV2Library.sol:54:        require(amountOut > 0, 'UniswapV2Library: INSUFFICIENT_OUTPUT_AMOUNT');
./libraries/UniswapV2Library.sol:55:        require(reserveIn > 0 && reserveOut > 0, 'UniswapV2Library: INSUFFICIENT_LIQUIDITY');
./libraries/UniswapV2Library.sol:77:        for (uint i = path.length - 1; i > 0; i--) {

```

