## Tags

- bug
- 2 (Med Risk)
- resolved
- sponsor confirmed

# [Staking: the rebase function needs to be called before calling the function in the Yieldy contract that uses the rebasingCreditsPerToken variable](https://github.com/code-423n4/2022-06-yieldy-findings/issues/126) 

# Lines of code

https://github.com/code-423n4/2022-06-yieldy/blob/524f3b83522125fb7d4677fa7a7e5ba5a2c0fe67/src/contracts/Staking.sol#L674-L696


# Vulnerability details

## Impact
In the Yieldy contract, functions such as balanceOf/creditsForTokenBalance/tokenBalanceForCredits/transfer/transferFrom/burn/mint will use the rebasingCreditsPerToken variable, so before calling these functions in the Staking contract, make sure that the rebase of this epoch has occurred. Therefore, the rebase function should also be called in the unstake/claim/claimWithdraw function of the Staking contract.
## Proof of Concept
https://github.com/code-423n4/2022-06-yieldy/blob/524f3b83522125fb7d4677fa7a7e5ba5a2c0fe67/src/contracts/Staking.sol#L674-L696
https://github.com/code-423n4/2022-06-yieldy/blob/524f3b83522125fb7d4677fa7a7e5ba5a2c0fe67/src/contracts/Staking.sol#L465-L508
## Tools Used
None
## Recommended Mitigation Steps
```
    function claim(address _recipient) public {
        Claim memory info = warmUpInfo[_recipient];
+      rebase();
...
    function claimWithdraw(address _recipient) public {
        Claim memory info = coolDownInfo[_recipient];
+      rebase();
...
    function unstake(uint256 _amount, bool _trigger) external {
        // prevent unstaking if override due to vulnerabilities asdf
        require(!isUnstakingPaused, "Unstaking is paused");
-        if (_trigger) {
            rebase();
-        }
```

