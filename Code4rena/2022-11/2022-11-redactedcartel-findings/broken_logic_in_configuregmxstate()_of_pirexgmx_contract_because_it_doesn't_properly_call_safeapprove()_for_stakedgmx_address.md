## Tags

- bug
- 2 (Med Risk)
- disagree with severity
- primary issue
- satisfactory
- selected for report
- sponsor confirmed
- M-09

# [broken logic in configureGmxState() of PirexGmx contract because it doesn't properly call safeApprove() for stakedGmx address](https://github.com/code-423n4/2022-11-redactedcartel-findings/issues/214) 

# Lines of code

https://github.com/code-423n4/2022-11-redactedcartel/blob/03b71a8d395c02324cb9fdaf92401357da5b19d1/src/PirexGmx.sol#L269-L293
https://github.com/code-423n4/2022-11-redactedcartel/blob/03b71a8d395c02324cb9fdaf92401357da5b19d1/src/PirexGmx.sol#L346-L355


# Vulnerability details

## Impact
Function `configureGmxState()` of `PirexGmx` is for configuring GMX contract state but logic is using `safeApprove()` improperly, it won't reset approval amount for old `stakedGmx` address This would cause 4 problem:
1. different behavior in `setContract()` and `configureGmxState()` for handling `stakeGmx` address changes. in `setContract()` the logic reset approval for old address to zero but in `configureGmxState()` the logic don't reset old address GMX spending approval.
2. the call to this function would revert if `stakeGmx` address didn't changed but other addresses has been changed so `owner` can't use this to configure contract.
3. contract won't reset approval for old `stakedGmx` addresse which is a threat because contract in that address can steal all the GMX balance any time in the future if that old address had been compromised.
4. contract won't reset approval for old `stakedGmx` addresse, if `owner` use `configureGmxState()` to change the `stakeGmx` value then it won't be possible to set `stakedGmx` value to previous ones by using either `configureGmxState()` or `setContract()` and contract would be in broken state.


## Proof of Concept
This is `configureGmxState()` code in `PirexGmx`:
```
    /**
        @notice Configure GMX contract state
     */
    function configureGmxState() external onlyOwner whenPaused {
        // Variables which can be assigned by reading previously-set GMX contracts
        rewardTrackerGmx = RewardTracker(gmxRewardRouterV2.feeGmxTracker());
        rewardTrackerGlp = RewardTracker(gmxRewardRouterV2.feeGlpTracker());
        feeStakedGlp = RewardTracker(gmxRewardRouterV2.stakedGlpTracker());
        stakedGmx = RewardTracker(gmxRewardRouterV2.stakedGmxTracker());
        glpManager = gmxRewardRouterV2.glpManager();
        gmxVault = IVault(IGlpManager(glpManager).vault());

        emit ConfigureGmxState(
            msg.sender,
            rewardTrackerGmx,
            rewardTrackerGlp,
            feeStakedGlp,
            stakedGmx,
            glpManager,
            gmxVault
        );

        // Approve GMX to enable staking
        gmx.safeApprove(address(stakedGmx), type(uint256).max);
    }
```
As you can see it just sets the approval for new `stakeGmx` address and don't do anything about spending approval of old `stakeGmx` address.
This is part of the `setContract()` code that handels `stakeGmx`:
```
        if (c == Contracts.StakedGmx) {
            // Set the current stakedGmx (pending change) approval amount to 0
            gmx.safeApprove(address(stakedGmx), 0);

            stakedGmx = RewardTracker(contractAddress);

            // Approve the new stakedGmx contract address allowance to the max
            gmx.safeApprove(contractAddress, type(uint256).max);
            return;
        }
```
As you can see it resets the spending approval of old `stakedGmx` address to zero and then give unlimited spending approval for new address.
So the impact #1 is obvious that the code for same logic in two different functions don't beehive similarly.

Function `configureGmxState()` is used for configuring GMX contract state but if `owner` uses this function one time then it won't be possible to call this function the second time if `stakedGmx` wasn't changed. for example in this scenario:
1. owner calls `configureGmxState()` to set values for GMX contract addresses.
2. then address of one of contract changes in GMX (`stakedGmx` stayed the same) and owner want's to call `configureGmxState()` to reset the values of variables to correct ones.
3. owner call to `configureGmxState()` would fail because `stakedGmx` address didn't changed and in the line ` gmx.safeApprove(address(stakedGmx), type(uint256).max);` contract tries to set non zero approval for `stakedGmx` but it already has non zero spending allowance. (`safeApprove()` would revert if the current spending allowance is not zero and the new allowance is not zero either).
so the impact #2 would happen and calls to this function in some scenarios would fail and it won't be functional.

Because function `configureGmxState()` don't reset the old `stakeGmx` address's GMX token spending approval to 0x0 so it would be possible to lose all GMX balance of `PirexGmx` contract if the old `stakeGmx` addresses are compromised. for example in this scenario:
1. GMX protocol get hacked(either by a bug or leaking some private keys) and `stakeGmx` contract control would be in hacker's hand.
2. GMX deploy new contracts and `stakeGmx` address changes in GMX. ()
3. owner of `PirexGmx` calls `configureGmxState()` to reset the values of GMX contracts addresses in `PirexGmx`. 
4. function `configureGmxState()` logic would change the GMX contract addresses but it won't set GMX token spending approval for old `stakeGmx` address to zero.
5. hacker who control the old `stakeGmx` address would use his access to that address contract to withdraw GMX balance of `PirexGmx`.
because `PirexGmx` won't set approval for old `stakeGmx` contract so it would be possible for that old `stakeGmx` address to transfer GMX balance of `PirexGmx` anytime in future. the bug in old `stakeGmx` contract or leakage of private keys of `stakeGmx` address(private key who can become the owner or admin of that contract) can be happen after migrating GMX and Pirex contracts too. This is impact #3 scenario.

in scenario #4 contract would stuck in unrecoverable state. the problem is that if `configureGmxState()` is get used more than once and `stakeGmx` variable's value has been changes more than once then it won't be possible to set `stakeGmx` value to old values either with `configureGmxState()` or `setContract()` and the contract won't be useful. the scenario is this: (`safeApprove()` won't allow to set non-zero approval for address that has already non-zero approval amount. see the OZ implementation)
1. GMX protocol changes its `stakeGmx` contract address from old address to new (for any reason, they got hacked or they are migrating or ....)
2. `owner` of `PirexGmx` calls `configureGmxState()` to update GMX protocol's contract address in `PirexGmx` contract and the logic would update the values of variables. (the GMX spending approval for old and new `stakeGmx` address would be max value).
3. GMX protocol changes `stakeGmx` contract address from new value to old value (for any reason, they decided to roll back to old address)
4. owner tries to call `configureGmxState()` to reupdate GMX protocol's address in `PirexGmx` but the call would revert because the code tries to call `safeApprove()` for address that has already non-zero allowance.
5. owner can't call `setContract()` to update value of `stakeGmx` variable because this function tries to call `safeApprove()` to set non-zero approval value for address that already has non-zero allowance.
so in this state `owner` can't recover `PirexGmx` contract and because contract has wrong value for `stakeGmx` it won't be functional and it would stay in broken state. 

## Tools Used
VIM

## Recommended Mitigation Steps
like `setContract()`, function `configureGmxState()` should set approval for old `PirexGmx` to zero first.