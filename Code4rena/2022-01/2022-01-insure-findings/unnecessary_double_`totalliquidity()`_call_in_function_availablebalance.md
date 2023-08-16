## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [unnecessary double `totalLiquidity()` call in function availableBalance](https://github.com/code-423n4/2022-01-insure-findings/issues/121) 

# Handle

Tomio


# Vulnerability details

## Impact
by saving `totalLiquidity()` to memory can save more gas instead of doing double function call

## Proof of Concept
Before: https://github.com/code-423n4/2022-01-insure/blob/main/contracts/PoolTemplate.sol#L829
// gas cost 23862

After:
```
function totalLiquidity() public view returns (uint256){
     return 10;
 }   

function availableBalance()public view returns (uint256 _balance)
    {
        uint256 saveTotalLiquidity = totalLiquidity();
        if (saveTotalLiquidity > 0) {
            return saveTotalLiquidity - lockedAmount;
        } else {
            return 0;
        }
    }
```
// gas cost 23840

## Tools Used
Remix

