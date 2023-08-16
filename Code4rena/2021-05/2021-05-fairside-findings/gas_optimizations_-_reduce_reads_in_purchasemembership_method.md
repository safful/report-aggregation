## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed
- resolved

# [Gas optimizations - Reduce reads in purchaseMembership method](https://github.com/code-423n4/2021-05-fairside-findings/issues/55) 

# Handle

a_delamo


# Vulnerability details

## Impact

The method `purchaseMembership` in `FSDNetwork` contract contains the code below.
Inside this method, we are constantly reading from the mapping `membership`, so why not use just one read `Membership userMembership = membership[msg.sender]` and use this instance for everything related to memberships. 
Each read we are currently doing has an impact on the gas cost.

```
function purchaseMembership(uint256 costShareBenefit) external {
        require(
            costShareBenefit % 10 ether == 0 && costShareBenefit > 0,
            "FSDNetwork::purchaseMembership: Invalid cost share benefit specified"
        );

        if (
            membership[msg.sender].creation + MEMBERSHIP_DURATION <
            block.timestamp
        ) {
            membership[msg.sender].creation = 0;
            membership[msg.sender].availableCostShareBenefits = 0;
        }

        uint256 totalCostShareBenefit =
            membership[msg.sender].availableCostShareBenefits.add(
                costShareBenefit
            );
        require(
            totalCostShareBenefit <= getMaximumBenefitPerUser(),
            "FSDNetwork::purchaseMembership: Exceeds cost share benefit limit per account"
        );

        totalCostShareBenefits = totalCostShareBenefits.add(costShareBenefit);

        // FSHARE = Total Available Cost Share Benefits / Gearing Factor
        uint256 fShare = totalCostShareBenefits / GEARING_FACTOR;
        // Floor of 7500 ETH
        if (fShare < 7500 ether) fShare = 7500 ether;

        // FSHARERatio = Capital Pool / FSHARE (scaled by 1e18)
        uint256 fShareRatio =
            (fsd.getReserveBalance() - totalOpenRequests).mul(1 ether) / fShare;

        // 1 ether = 100%
        require(
            fShareRatio >= 1 ether,
            "FSDNetwork::purchaseMembership: Insufficient Capital to Cover Membership"
        );

        uint256 membershipFee = costShareBenefit.wmul(MEMBERSHIP_FEE);
        uint256 fsdSpotPrice = getFSDPrice();
        uint256 fsdFee = membershipFee.wdiv(fsdSpotPrice);

        // Automatically locks 65% to the Network by disallowing its retrieval
        fsd.safeTransferFrom(msg.sender, address(this), fsdFee);

        if (membership[msg.sender].creation == 0) {
            membership[msg.sender]
                .availableCostShareBenefits = totalCostShareBenefit;
            membership[msg.sender].creation = block.timestamp;
            membership[msg.sender].gracePeriod =
                membership[msg.sender].creation +
                MEMBERSHIP_DURATION +
                60 days;
        } else {
            membership[msg.sender]
                .availableCostShareBenefits = totalCostShareBenefit;

            uint256 elapsedDurationPercentage =
                ((block.timestamp - membership[msg.sender].creation) *
                    1 ether) / MEMBERSHIP_DURATION;
            if (elapsedDurationPercentage < 1 ether) {
                uint256 durationIncrease =
                    (costShareBenefit.mul(1 ether) /
                        (totalCostShareBenefit - costShareBenefit))
                        .mul(MEMBERSHIP_DURATION) / 1 ether;
                membership[msg.sender].creation += durationIncrease;
            }
        }

        uint256 governancePoolRewards =
            fsdFee.wmul(GOVERNANCE_FUNDING_POOL_REWARDS);

        // Staking Rewards = 20% + [FSHARERatio - 125%] (if FSHARERatio > 125%)
        uint256 stakingMultiplier =
            fShareRatio >= 1.25 ether
                ? STAKING_REWARDS + fShareRatio - 1.25 ether
                : STAKING_REWARDS;

        // Maximum of 75% as we have 15% distributed to governance + funding pool
        if (stakingMultiplier > 0.75 ether) stakingMultiplier = 0.75 ether;

        uint256 stakingRewards = fsdFee.wmul(stakingMultiplier);

        // 20% as staking rewards
        fsd.safeTransfer(address(fsd), stakingRewards);
        fsd.addRegistrationTribute(stakingRewards);

        // 7.5% towards governance
        fsd.safeTransfer(address(fsd), governancePoolRewards);
        fsd.addRegistrationTributeGovernance(governancePoolRewards);

        // 7.5% towards funding pool
        fsd.safeTransfer(FUNDING_POOL, governancePoolRewards);
    }
```

