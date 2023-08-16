## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Prefix increments are cheaper than postfix increments](https://github.com/code-423n4/2021-12-yetifinance-findings/issues/12) 

# Handle

robee


# Vulnerability details

Prefix increments are cheaper than postfix increments. 
Further more, using unchecked {++x} is even more gas efficient, and the gas saving accumulates every iteration and can make a real change
There is no risk of overflow caused by increamenting the iteration index in for loops (the `++i` in `for (uint256 i = 0; i < numIterations; ++i)`).
But increments perform overflow checks that are not necessary in this case.
These functions use not using prefix increments (`++x`) or not using the unchecked keyword: 

        change to prefix increment and unchecked: ActivePool.sol, i, 133
        change to prefix increment and unchecked: ActivePool.sol, i, 167
        change to prefix increment and unchecked: ActivePool.sol, i, 184
        change to prefix increment and unchecked: BorrowerOperations.sol, i, 699
        change to prefix increment and unchecked: BorrowerOperations.sol, i, 737
        change to prefix increment and unchecked: BorrowerOperations.sol, i, 873
        change to prefix increment and unchecked: BorrowerOperations.sol, i, 890
        change to prefix increment and unchecked: BorrowerOperations.sol, i, 920
        change to prefix increment and unchecked: BorrowerOperations.sol, i, 929
        change to prefix increment and unchecked: BorrowerOperations.sol, i, 1061
        change to prefix increment and unchecked: BorrowerOperations.sol, i, 1068
        change to prefix increment and unchecked: DefaultPool.sol, i, 97
        change to prefix increment and unchecked: DefaultPool.sol, i, 133
        change to prefix increment and unchecked: LiquityBase.sol, i, 63
        change to prefix increment and unchecked: LiquityBase.sol, i, 97
        change to prefix increment and unchecked: LiquityBase.sol, i, 106
        change to prefix increment and unchecked: LiquityBase.sol, i, 115
        change to prefix increment and unchecked: LiquityBase.sol, i, 149
        change to prefix increment and unchecked: LiquityBase.sol, i, 158
        change to prefix increment and unchecked: LiquityBase.sol, i, 168
        change to prefix increment and unchecked: YetiCustomBase.sol, i, 36
        change to prefix increment and unchecked: YetiCustomBase.sol, i, 44
        change to prefix increment and unchecked: YetiCustomBase.sol, i, 59
        change to prefix increment and unchecked: YetiCustomBase.sol, i, 104
        change to prefix increment and unchecked: YetiCustomBase.sol, i, 122
        change to prefix increment and unchecked: YetiCustomBase.sol, i, 144
        change to prefix increment and unchecked: YetiCustomBase.sol, i, 152
        change to prefix increment and unchecked: YetiCustomBase.sol, i, 165
        change to prefix increment and unchecked: YetiCustomBase.sol, i, 178
        change to prefix increment and unchecked: HintHelpers.sol, i, 139
        just change to unchecked: MultiTroveGetter.sol, idx, 73
        just change to unchecked: MultiTroveGetter.sol, idx, 79
        just change to unchecked: MultiTroveGetter.sol, idx, 90
        just change to unchecked: MultiTroveGetter.sol, idx, 96
        change to prefix increment and unchecked: MultiTroveGetter.sol, i, 110
        change to prefix increment and unchecked: StabilityPool.sol, i, 562
        change to prefix increment and unchecked: StabilityPool.sol, i, 588
        change to prefix increment and unchecked: StabilityPool.sol, i, 592
        change to prefix increment and unchecked: StabilityPool.sol, i, 628
        change to prefix increment and unchecked: StabilityPool.sol, i, 720
        change to prefix increment and unchecked: StabilityPool.sol, i, 942
        change to prefix increment and unchecked: StabilityPool.sol, i, 1011
        change to prefix increment and unchecked: TeamAllocation.sol, i, 66
        change to prefix increment and unchecked: CDPManagerTester.sol, i, 14
        change to prefix increment and unchecked: EchidnaTester.sol, i, 103
        change to prefix increment and unchecked: EchidnaTester.sol, i, 121
        change to prefix increment and unchecked: TroveManager.sol, i, 234
        change to prefix increment and unchecked: TroveManager.sol, i, 348
        change to prefix increment and unchecked: TroveManager.sol, i, 374
        change to prefix increment and unchecked: TroveManager.sol, i, 397
        change to prefix increment and unchecked: TroveManager.sol, i, 420
        change to prefix increment and unchecked: TroveManager.sol, i, 460
        change to prefix increment and unchecked: TroveManager.sol, i, 476
        change to prefix increment and unchecked: TroveManager.sol, i, 525
        change to prefix increment and unchecked: TroveManager.sol, i, 582
        change to prefix increment and unchecked: TroveManager.sol, i, 603
        change to prefix increment and unchecked: TroveManagerLiquidations.sol, vars.i, 255
        change to prefix increment and unchecked: TroveManagerLiquidations.sol, vars.i, 334
        change to prefix increment and unchecked: TroveManagerLiquidations.sol, i, 394
        change to prefix increment and unchecked: TroveManagerLiquidations.sol, i, 475
        change to prefix increment and unchecked: TroveManagerLiquidations.sol, i, 808
        change to prefix increment and unchecked: TroveManagerLiquidations.sol, i, 840
        change to prefix increment and unchecked: TroveManagerRedemptions.sol, i, 304
        change to prefix increment and unchecked: TroveManagerRedemptions.sol, i, 517

