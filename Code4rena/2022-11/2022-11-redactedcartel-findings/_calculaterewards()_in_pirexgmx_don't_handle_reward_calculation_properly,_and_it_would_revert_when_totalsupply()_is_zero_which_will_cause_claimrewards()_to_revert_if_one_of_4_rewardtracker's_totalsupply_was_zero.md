## Tags

- bug
- 2 (Med Risk)
- primary issue
- satisfactory
- selected for report
- sponsor confirmed
- M-10

# [_calculateRewards() in PirexGmx don't handle reward calculation properly, and it would revert when totalSupply() is zero which will cause claimRewards() to revert if one of 4 rewardTracker's totalSupply was zero](https://github.com/code-423n4/2022-11-redactedcartel-findings/issues/237) 

# Lines of code

https://github.com/code-423n4/2022-11-redactedcartel/blob/03b71a8d395c02324cb9fdaf92401357da5b19d1/src/PirexGmx.sol#L733-L816
https://github.com/code-423n4/2022-11-redactedcartel/blob/03b71a8d395c02324cb9fdaf92401357da5b19d1/src/PirexGmx.sol#L228-L267
https://github.com/code-423n4/2022-11-redactedcartel/blob/03b71a8d395c02324cb9fdaf92401357da5b19d1/src/PirexRewards.sol#L332-L348


# Vulnerability details

## Impact
Function `claimRewards()` in `PirexGmx` claims WETH and esGMX rewards and multiplier points (MP) from GMX protocol. it uses `_calculateRewards()` to calculate the unclaimed reward token amounts produced for each token type. but because of the lack of checks, function `_calculateRewards()` would revert when `RewardTracker.distributor.totalSupply()` is zero so if any of 4 `RewardTracker`s has zero `totalSupply()` then function `claimRewards()` would revert too and function `harvest()` in `PirexRewards` contract would revert too because it calls `PirexGmx.claimRewards()`. `harvest()` is used in `claim()` function so `claim()` would not work too. This bug would harvesting rewards for `PirexRewards` contract and claiming reward for users from `PirexRewards` when supply of one of `RewardTracker`s contracts in GMX protocol is zero.
Function `claimRewards()` is written based on GMX code, but the logic is not correctly copied because in GMX protocol contract checks for `totalSupply()` and it prevents this bug from happening.

## Proof of Concept
This is function `_calculateRewards()`'s code in `PirexGmx`:
```
   function _calculateRewards(bool isBaseReward, bool useGmx)
        internal
        view
        returns (uint256)
    {
        RewardTracker r;

        if (isBaseReward) {
            r = useGmx ? rewardTrackerGmx : rewardTrackerGlp;
        } else {
            r = useGmx ? stakedGmx : feeStakedGlp;
        }

        address distributor = r.distributor();
        uint256 pendingRewards = IRewardDistributor(distributor)
            .pendingRewards();
        uint256 distributorBalance = (isBaseReward ? gmxBaseReward : esGmx)
            .balanceOf(distributor);
        uint256 blockReward = pendingRewards > distributorBalance
            ? distributorBalance
            : pendingRewards;
        uint256 precision = r.PRECISION();
        uint256 cumulativeRewardPerToken = r.cumulativeRewardPerToken() +
            ((blockReward * precision) / r.totalSupply());

        if (cumulativeRewardPerToken == 0) return 0;

        return
            r.claimableReward(address(this)) +
            ((r.stakedAmounts(address(this)) *
                (cumulativeRewardPerToken -
                    r.previousCumulatedRewardPerToken(address(this)))) /
                precision);
    }
```
As you can see in the line `uint256 cumulativeRewardPerToken = r.cumulativeRewardPerToken() + ((blockReward * precision) / r.totalSupply())` if `totalSupply()` was zero then code would revert because of division by zero error. so if `RewardTracker.distributor.totalSupply()` was zero then function `_calculateRewards` would revert and won't work and other function using `_calculateRewards()` would be break too. 
This is part of function `claimRewards()`'s code in `PirexGmx` contract:
```
    function claimRewards()
        external
        onlyPirexRewards
        returns (
            ERC20[] memory producerTokens,
            ERC20[] memory rewardTokens,
            uint256[] memory rewardAmounts
        )
    {
        // Assign return values used by the PirexRewards contract
        producerTokens = new ERC20[](4);
        rewardTokens = new ERC20[](4);
        rewardAmounts = new uint256[](4);
        producerTokens[0] = pxGmx;
        producerTokens[1] = pxGlp;
        producerTokens[2] = pxGmx;
        producerTokens[3] = pxGlp;
        rewardTokens[0] = gmxBaseReward;
        rewardTokens[1] = gmxBaseReward;
        rewardTokens[2] = ERC20(pxGmx); // esGMX rewards distributed as pxGMX
        rewardTokens[3] = ERC20(pxGmx);

        // Get pre-reward claim reward token balances to calculate actual amount received
        uint256 baseRewardBeforeClaim = gmxBaseReward.balanceOf(address(this));
        uint256 esGmxBeforeClaim = stakedGmx.depositBalances(
            address(this),
            address(esGmx)
        );

        // Calculate the unclaimed reward token amounts produced for each token type
        uint256 gmxBaseRewards = _calculateRewards(true, true);
        uint256 glpBaseRewards = _calculateRewards(true, false);
        uint256 gmxEsGmxRewards = _calculateRewards(false, true);
        uint256 glpEsGmxRewards = _calculateRewards(false, false);
```
As you can see it calls `_calculateRewards()` to calculate  the unclaimed reward token amounts  produced for each token type in GMX protocol. so function `claimRewards()` would revert too when `totalSupply()` of one of these 4 `RewardTracker`'s distributers was zero.
This is part of functions `harvest()` and `claim()` code in `PirexReward` contract:
```
    function harvest()
        public
        returns (
            ERC20[] memory _producerTokens,
            ERC20[] memory rewardTokens,
            uint256[] memory rewardAmounts
        )
    {
        (_producerTokens, rewardTokens, rewardAmounts) = producer
            .claimRewards();
        uint256 pLen = _producerTokens.length;
.......
......
......


    function claim(ERC20 producerToken, address user) external {
        if (address(producerToken) == address(0)) revert ZeroAddress();
        if (user == address(0)) revert ZeroAddress();

        harvest();
        userAccrue(producerToken, user);
....
....
....
```
As you can see `harvest()` calls `claimRewards()` and `claim()` calls `harvest()` so these two function would revert and won't work when `totalSupply()` of one of these 4 `RewardTracker`'s distributers in GMX protocol was zero. in that situation the protocol can't harvest and claim rewards from GMX because of this bug and users won't be able to claim their rewards from the protocol. the condition for this bug could happen from time to time as GMX decided to prevent it by checking the value of `totalSupply()`. This is function `_updateRewards()` code in `RewardTracker` in GMX protocol (https://github.com/gmx-io/gmx-contracts/blob/65e62b62aadea5baca48b8157acb9351249dbaf1/contracts/staking/RewardTracker.sol#L272-L286):
```
    function _updateRewards(address _account) private {
        uint256 blockReward = IRewardDistributor(distributor).distribute();

        uint256 supply = totalSupply;
        uint256 _cumulativeRewardPerToken = cumulativeRewardPerToken;
        if (supply > 0 && blockReward > 0) {
            _cumulativeRewardPerToken = _cumulativeRewardPerToken.add(blockReward.mul(PRECISION).div(supply));
            cumulativeRewardPerToken = _cumulativeRewardPerToken;
        }

        // cumulativeRewardPerToken can only increase
....
....
```
As you can see it checks that `supply > 0` before using it as denominator in division. So GMX protocol handles the case when `totalSupply()` is zero and contract logics won't break when this case happens but function `_calculateRewards()` which tries to calculate GMX protocol rewards beforehand don't handle this case(the case where `totalSupply()` is zero) so the logics would break when this case happens and it would cause function `harvest()` and `claim()` to be unfunctional.

## Tools Used
VIM

## Recommended Mitigation Steps
check that `totalSupply()` is not zero before using it.