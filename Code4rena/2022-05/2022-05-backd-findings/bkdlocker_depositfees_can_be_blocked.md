## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [BkdLocker depositFees can be blocked](https://github.com/code-423n4/2022-05-backd-findings/issues/8) 

# Lines of code

https://github.com/code-423n4/2022-05-backd/blob/main/protocol/contracts/RewardHandler.sol#L50


# Vulnerability details

## Impact
burnFees will fail if none of the pool tokens have underlying token as native ETH token. This is shown below. Since burnFees fails so no fees is deposited in BKDLocker

## Proof of Concept
1. Assume RewardHandler.sol has currently amount 5 as address(this).balance (ethBalance) (even attacker can send a small balance to this contract to do this dos attack)
2. None of the pools have underlying as address(0) so no ETH tokens and only ERC20 tokens are present
3. Now feeBurner.burnToTarget is called passing current ETH balance of amount 5 with all pool tokens
4. feeBurner loops through all tokens and swap them to WETH. Since none of the token is ETH so burningEth_ variable is false
5. Now the below require condition fails since burningEth_ is false 

```
require(burningEth_ || msg.value == 0, Error.INVALID_VALUE);
```

6. This fails the burnFees function.

## Recommended Mitigation Steps
ETH should not be sent if none of pool underlying token is ETH. Change it to something like below:

```
bool ethFound=false;
for (uint256 i; i < pools.length; i = i.uncheckedInc()) {
            ILiquidityPool pool = ILiquidityPool(pools[i]);
            address underlying = pool.getUnderlying();
            if (underlying != address(0)) {
                _approve(underlying, address(feeBurner));
            } else
{
ethFound=true;
}
            tokens[i] = underlying;
        }

if(ethFound){
        feeBurner.burnToTarget{value: ethBalance}(tokens, targetLpToken);
} else {
feeBurner.burnToTarget(tokens, targetLpToken);
}
```

