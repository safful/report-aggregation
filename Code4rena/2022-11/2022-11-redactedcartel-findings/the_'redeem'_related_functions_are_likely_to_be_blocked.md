## Tags

- bug
- 3 (High Risk)
- primary issue
- selected for report
- sponsor confirmed
- edited-by-warden
- H-01

# [The 'redeem' related functions are likely to be blocked](https://github.com/code-423n4/2022-11-redactedcartel-findings/issues/113) 

# Lines of code

https://github.com/code-423n4/2022-11-redactedcartel/blob/03b71a8d395c02324cb9fdaf92401357da5b19d1/src/PirexGmx.sol#L615
https://github.com/code-423n4/2022-11-redactedcartel/blob/03b71a8d395c02324cb9fdaf92401357da5b19d1/src/PirexGmx.sol#L685
https://github.com/code-423n4/2022-11-redactedcartel/blob/03b71a8d395c02324cb9fdaf92401357da5b19d1/src/PirexGmx.sol#L712


# Vulnerability details

## Impact
The following 'redeem' related functions are likely to be blocked, users will not be able to retrieve their funds.
```
function _redeemPxGlp(
    address token,
    uint256 amount,
    uint256 minOut,
    address receiver
);
function redeemPxGlpETH(
    uint256 amount,
    uint256 minOut,
    address receiver
);
function redeemPxGlp(
    address token,
    uint256 amount,
    uint256 minOut,
    address receiver
);
```

## Proof of Concept
The 'GlpManager' contract of GMX has a 'cooldownDuration' limit on redeem/unstake ('_removeLiquidity()'). While there is at least one deposit/stake ('_addLiquidity()') operation in the past 'cooldownDuration' time, redemption would fail. Obviously this limitation is user-based,  and 'PirexGmx' contract is one such user.

https://github.com/gmx-io/gmx-contracts/blob/c3618b0d6fc1b88819393dc7e6c785e32e78c72b/contracts/core/GlpManager.sol#L234

```
Current setting of 'cooldownDuration' is 15 minutes, the max value is 2 days.
```
https://arbiscan.io/address/0x321f653eed006ad1c29d174e17d96351bde22649#readContract

Due to the above limit, there are 3 risks that can block redemption for Pirex users.

1.__The normal case__
Let's say there is 10% GMX users will use Pirex to manage their GLP. 
By checking recent history of GMX router contract, we can find the average stake interval is smaller than 1 minute
https://arbiscan.io/address/0xa906f338cb21815cbc4bc87ace9e68c87ef8d8f1
Let's take
```
averageStakeIntervalOfGMX = 30 seconds
```
So if Pirex has 10% of GMX users, then
```
averageStakeIntervalOfPirex = 30 ÷ 10% = 300 seconds
```
The probability of successfully redeeming is a typical Poisson distribution: https://en.wikipedia.org/wiki/Poisson_distribution
With
```
λ = cooldownDuration ÷ averageStakeIntervalOfPirex = 15 * 60 ÷ 300 = 3
k = 0
```
So we get
```
P ≈ 1 ÷ (2.718 * 2.718 * 2.718) ≈ 0.05 
```
Conclusion
```
If Pirex has 10 % of GMX users, then the redemption will fail with 95% probability.
```

A full list of % of GMX users versus failure probability of redemption
```
1% : 26%
5% : 78%
10% : 95%
20% : 99.75%
30% : 99.98%
```

2.__The attack case__
If an attacker, such as bad competitors of similar projects, try to exploit this vulnerability. Let's estimate the cost for attack.

As attacker can deposit a very small GLP, such as 1 wei, so we can ignore the GLP cost and only focus on GAS cost.

By checking the explorer history https://arbiscan.io
We are safe to assume the cost for calling 'depositGlpETH()' or 'depositGlp' is
```
txCost = 0.1 USD
```

To block redemption, attacker have to execute a deposit call every 15 minutes, so
```
dailyCost = 24 * (60 / 15) * 0.1 = 9.6 USD
yearCost = 365 * 9.6 = 3504 USD
```
Conclusion
```
If an attacker want to block Pirex users funds, his yearly cost is only about 3.5k USD.
```

3.__GMX adjusts protocol parameters__
If GMX increases 'cooldownDuration' to 2 days, it will obviously cause redemption not working.

## Tools Used
VS Code

## Recommended Mitigation Steps
Reserve some time range for redemption only. e.g. 1 of every 7 days.