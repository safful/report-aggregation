## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [`ConvexStakingWrapper.enterShelter()` May Erroneously Overwrite `amountInShelter` Leading To Locked Tokens](https://github.com/code-423n4/2022-02-concur-findings/issues/109) 

# Lines of code

https://github.com/code-423n4/2022-02-concur/blob/shelter-client/contracts/ConvexStakingWrapper.sol#L107-L119
https://github.com/code-423n4/2022-02-concur/blob/shelter-client/contracts/ConvexStakingWrapper.sol#L132-L135


# Vulnerability details

## Impact

The shelter mechanism provides emergency functionality in an effort to protect users' funds. The `enterShelter` function will withdraw all LP tokens from the pool, transfer them to the shelter contract and activate the shelter for the target LP token. If this function is called again on the same LP token, the `amountInShelter` value is overwritten, potentially by the zero amount. As a result  its possible that the shelter is put in a state where no users can withdraw from it or only a select few users with a finite number of shares are able to. Once the shelter has passed its grace period, these tokens may forever be locked in the shelter contract.

## Proof of Concept

https://github.com/code-423n4/2022-02-concur/blob/shelter-client/contracts/ConvexStakingWrapper.sol#L107-L119
```
function enterShelter(uint256[] calldata _pids) external onlyOwner {
    for(uint256 i = 0; i<_pids.length; i++){
        IRewardStaking pool = IRewardStaking(convexPool[_pids[i]]);
        uint256 amount = pool.balanceOf(address(this));
        pool.withdrawAndUnwrap(amount, false);
        IERC20 lpToken = IERC20(
            pool.poolInfo(_pids[i]).lptoken
        );
        amountInShelter[lpToken] = amount;
        lpToken.safeTransfer(address(shelter), amount);
        shelter.activate(lpToken);
    }
}
```

https://github.com/code-423n4/2022-02-concur/blob/shelter-client/contracts/ConvexStakingWrapper.sol#L132-L135
```
function totalShare(IERC20 _token) external view override returns(uint256) {
    // this will be zero if shelter is not activated
    return amountInShelter[_token];
}
```

## Tools Used

Manual code review.

## Recommended Mitigation Steps

Consider adding to the `amountInShelter[lpToken]` mapping instead of overwriting it altogether. This will allow `enterShelter` to be called multiple times with no loss of funds for the protocol's users.

