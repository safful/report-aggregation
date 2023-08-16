## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Remove extra calls in updateReserve (FeePoolV0.sol)](https://github.com/code-423n4/2021-10-mochi-findings/issues/79) 

# Handle

ye0lde


# Vulnerability details

## Impact

Cache the result of engine.usdm().balanceOf to simplify code and save gas.

## Proof of Concept

engine.usdm().balanceOf is called twice in function updateReserve here:
https://github.com/code-423n4/2021-10-mochi/blob/8458209a52565875d8b2cefcb611c477cefb9253/projects/mochi-core/contracts/feePool/FeePoolV0.sol#L32-L38

I suggest modifying the code as follows:
<code>
function updateReserve() external override { 
	uint256 balanceOf = engine.usdm().balanceOf(address(this));
	treasuryShare += ((balanceOf - mochiShare - treasuryShare) * treasuryRatio) / 1e18;
	mochiShare = balanceOf - treasuryShare;
}
</code>

## Tools Used
Visual Studio Code, Remix

## Recommended Mitigation Steps
See POC


