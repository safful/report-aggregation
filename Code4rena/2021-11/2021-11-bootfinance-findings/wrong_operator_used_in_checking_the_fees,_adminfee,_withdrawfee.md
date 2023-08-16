## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [wrong operator used in checking the fees, adminfee, withdrawfee](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/254) 

# Handle

JMukesh


# Vulnerability details

## Impact
wrong operator used in checking the fees, adminfee, withdrawfee instead of 

   require(_fee < SwapUtils.MAX_SWAP_FEE, "_fee exceeds maximum");

     _fee < = SwapUtils.Max_Swap_Fee , should be there same with adminfee & withdrawfee becuase in using <= it does not exceed the max value




## Proof of Concept
https://github.com/code-423n4/2021-11-bootfinance/blob/b4ebd0a5ebcbc24f3d15836cdb9759243fc85868/customswap/contracts/Swap.sol#L192


## Tools Used
manual review

## Recommended Mitigation Steps
use correct operator to check the value

