## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Yield distribution after large payout seems unfair](https://github.com/code-423n4/2021-07-sherlock-findings/issues/50) 

# Handle

gpersoon


# Vulnerability details

## Impact
When a large payout occurs, it will lower unallocatedSherX. This could mean some parties might not be able to get their Yield.

The first couple of users (for which harvest is called or which transfer tokens) will be able to get their full Yield, 
until the moment unallocatedSherX is depleted.
The next users don't get any yield at all.
This doesn't seem fair.

## Proof of Concept
// https://github.com/code-423n4/2021-07-sherlock/blob/main/contracts/facets/SherX.sol#L309
function doYield(ILock token,address from, address to, uint256 amount) private {
...
ps.unallocatedSherX = ps.unallocatedSherX.sub(withdrawable_amount);

//https://github.com/code-423n4/2021-07-sherlock/blob/main/contracts/facets/Payout.sol#L108
 function payout( address _payout, IERC20[] memory _tokens, uint256[] memory _firstMoneyOut, uint256[] memory _amounts, uint256[] memory _unallocatedSherX,  address _exclude ) external override onlyGovPayout {
    // all pools (including SherX pool) can be deducted fmo and balance
    // deducting balance will reduce the users underlying value of stake token
    // for every pool, _unallocatedSherX can be deducted, this will decrease outstanding SherX rewards
    // for users that did not claim them (e.g materialized them and included in SherX pool)
....
    // Subtract from unallocated, as the tokens are now allocated to this payout call
        ps.unallocatedSherX = ps.unallocatedSherX.sub(unallocatedSherX);

## Tools Used

## Recommended Mitigation Steps
If unallocatedSherX is insufficient to provide for all the yields, only give the yields partly (so that each user gets their fair share).


