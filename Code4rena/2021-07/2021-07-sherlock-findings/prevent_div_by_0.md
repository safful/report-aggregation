## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [prevent div by 0](https://github.com/code-423n4/2021-07-sherlock-findings/issues/22) 

# Handle

gpersoon


# Vulnerability details

## Impact
On several locations in the code precautions are taken not to divide by 0, because this will revert the code.
However on some locations this isn't done.

Especially in doYield a first check is done for totalAmount >0, however a few lines later there is an other div(totalAmount) which isn't checked.

The proof of concept show another few examples.

## Proof of Concept
// https://github.com/code-423n4/2021-07-sherlock/blob/main/contracts/facets/SherX.sol#L309
function doYield(ILock token,address from,address to,uint256 amount) private {
..
    uint256 totalAmount = ps.lockToken.totalSupply();
.. 
    if (totalAmount > 0) {
      ineglible_yield_amount = ps.sWeight.mul(amount).div(totalAmount);
    } else {
      ineglible_yield_amount = amount;
    }
    if (from != address(0)) {
      uint256 raw_amount = ps.sWeight.mul(userAmount).div(totalAmount);  // totalAmount could be 0, see lines above

// https://github.com/code-423n4/2021-07-sherlock/blob/main/contracts/facets/PoolBase.sol#L295
function activateCooldown(uint256 _amount, IERC20 _token) external override returns (uint256) {
...   uint256 tokenAmount = fee.mul(LibPool.stakeBalance(ps)).div(ps.lockToken.totalSupply());   // ps.lockToken.totalSupply() might be 0

//https://github.com/code-423n4/2021-07-sherlock/blob/main/contracts/facets/PoolBase.sol#L351
 function unstake( uint256 _id, address _receiver, IERC20 _token ) external override returns (uint256 amount) {
...    amount = withdraw.lock.mul(LibPool.stakeBalance(ps)).div(ps.lockToken.totalSupply());  // // ps.lockToken.totalSupply() might be 0

//https://github.com/code-423n4/2021-07-sherlock/blob/main/contracts/libraries/LibPool.sol#L67
 function stake( PoolStorage.Base storage ps,uint256 _amount, address _receiver ) external returns (uint256 lock) {
...      lock = _amount.mul(totalLock).div(stakeBalance(ps));   // stakeBalance(ps) might be 0

## Tools Used

## Recommended Mitigation Steps
Make sure division by 0 won't occur by checking the variables beforehand and handling this edge case.

