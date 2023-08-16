## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Reduce State Variable Use in VestedRewardPool.sol](https://github.com/code-423n4/2021-10-mochi-findings/issues/43) 

# Handle

ye0lde


# Vulnerability details

## Impact

Caching the "vesting" state variable instead of repeatedly reading and writing it will decrease deployment and runtime gas.  This is especially true for the modifier "checkClaimable" which is used on every function in the contract. 

## Proof of Concept

The checkClaimable function is here:
https://github.com/code-423n4/2021-10-mochi/blob/8458209a52565875d8b2cefcb611c477cefb9253/projects/mochi-core/contracts/emission/VestedRewardPool.sol#L22-L29

An example of its use is here along with many other accesses to the "vesting" state variable.
https://github.com/code-423n4/2021-10-mochi/blob/8458209a52565875d8b2cefcb611c477cefb9253/projects/mochi-core/contracts/emission/VestedRewardPool.sol#L36-L46

## Tools Used
Visual Studio Code, Remix

## Recommended Mitigation Steps

I suggest modifying "checkClaimable as follows: 
  
<code>
function checkClaimable(Vesting memory v) internal pure returns(Vesting memory) { 
	if (v.ends < block.timestamp) {
		v.claimable += v.vested;
		v.vested = 0;
		v.ends = 0;
	}
	return v;
}
</code>

and I suggest these changes to function "vest" 

<code>
function vest(address _recipient) external {
	Vesting memory v = checkClaimable(vesting[_recipient]);
	uint256 amount = mochi.balanceOf(address(this)) - mochiUnderManagement;
	uint256 weightedEnd = (v.vested *
		v.ends +
		amount *
		(block.timestamp + 90 days)) /
		(v.vested + amount);
	v.vested += amount;
	v.ends = weightedEnd;
	vesting[_recipient] = v;
	mochiUnderManagement += amount;
}
</code>

These functions are also candidates for similar changes:
https://github.com/code-423n4/2021-10-mochi/blob/8458209a52565875d8b2cefcb611c477cefb9253/projects/mochi-core/contracts/emission/VestedRewardPool.sol#L48-L71


