## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [Unable To Call `emergencyWithdraw` ETH in `NoYield` Contract](https://github.com/code-423n4/2021-12-sublime-findings/issues/52) 

# Handle

leastwood


# Vulnerability details

## Impact

The `emergencyWithdraw` function is implemented in all yield sources to allow the `onlyOwner` role to drain the contract's balance in case of emergency. The contract considers ETH as a zero address asset. However, there is a call made on `_asset` which will revert if it is the zero address. As a result, ETH tokens can never be withdrawn from the `NoYield` contract in the event of an emergency.

## Proof of Concept

Consider the case where `_asset == address(0)`. An external call is made to check the contract's token balance for the target `_asset`. However, this call will revert as `_asset` is the zero address. As a result, the `onlyOwner` role will never be able to withdraw ETH tokens during an emergency.

```
function emergencyWithdraw(address _asset, address payable _wallet) external onlyOwner returns (uint256 received) {
    require(_wallet != address(0), 'cant burn');
    uint256 amount = IERC20(_asset).balanceOf(address(this));
    IERC20(_asset).safeTransfer(_wallet, received);
    received = amount;
}
```

Affected function as per below:
https://github.com/code-423n4/2021-12-sublime/blob/main/contracts/yield/NoYield.sol#L78-L83

## Tools Used

Manual code review.

## Recommended Mitigation Steps

Consider handling the case where `_asset` is the zero address, i.e. the asset to be withdrawn under emergency is the ETH token.

