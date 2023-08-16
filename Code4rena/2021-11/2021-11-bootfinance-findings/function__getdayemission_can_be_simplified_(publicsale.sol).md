## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed

# [Function _getDayEmission can be simplified (PublicSale.sol)](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/118) 

# Handle

ye0lde


# Vulnerability details

## Impact

Code clarity

## Proof of Concept

The function is here:
https://github.com/code-423n4/2021-11-bootfinance/blob/7c457b2b5ba6b2c887dafdf7428fd577e405d652/tge/contracts/PublicSale.sol#L291-L298

## Tools Used
Visual Studio Code, Remix

## Recommended Mitigation Steps

I suggest making the changes below to simplify:
  
<code>
    function getDayEmission() public view returns (uint) {   
        return (remainingSupply > emission ? emission : remainingSupply);
    }
</code>

