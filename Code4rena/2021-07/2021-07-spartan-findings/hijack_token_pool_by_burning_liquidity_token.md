## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [Hijack token pool by burning liquidity token](https://github.com/code-423n4/2021-07-spartan-findings/issues/38) 

# Handle

jonah1005


# Vulnerability details

## Impact

`Pool` allows users to burn lp tokens without withdrawing the tokens. This allows the hacker to mutate the pools' rate to a point that no one can get any lp token anymore (even if depositing token).

The liquidity tokens are calculated at `Utils:calcLiquidityUnits`
```
            // units = ((P (t B + T b))/(2 T B)) * slipAdjustment
            // P * (part1 + part2) / (part3) * slipAdjustment
            uint slipAdjustment = getSlipAdustment(b, B, t, T);
            uint part1 = t*(B);
            uint part2 = T*(b);
            uint part3 = T*(B)*(2);
            uint _units = (P * (part1 + (part2))) / (part3);
            return _units * slipAdjustment / one;  // Divide by 10**18
```
where `P` stands for `totalSupply` of current Pool. If `P` is too small (e.g, 1) then all the units would be rounding to 0.

Since any person can create a `Pool` at `PoolFactory`, hackers can create a Pool and burn his lp and set `totalSupply` to 1. He will be the only person who owns the Pool's lp from now on.

## Proof of Concept
Pool's burn logic:
https://github.com/code-423n4/2021-07-spartan/blob/e2555aab44d9760fdd640df9095b7235b70f035e/contracts/Pool.sol#L146

Utils' lp token formula:
https://github.com/code-423n4/2021-07-spartan/blob/e2555aab44d9760fdd640df9095b7235b70f035e/contracts/Utils.sol#L80

Here's a script of a user depositing 1M token to a pool where `totalSupply` equals 1

```
dai_pool.functions.burn(init_amount-1).transact()
print('total supply', dai_pool.functions.totalSupply().call())
dai.functions.transfer(dai_pool.address, 1000000 * 10**18).transact()
dai_pool.functions.addForMember(user).transact()
print('lp received from depositing 1M dai: ', dai_pool.functions.balanceOf(user).call())
```

Output:
```
total supply 1
lp received from depositing 1M dai:  0
```
## Tools Used
None
## Recommended Mitigation Steps
Remove `burn` or restrict it to privileged users only.


