## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [YearnTokenAdapter's wrap can become stuck as it uses one step approval for an arbitrary underlying](https://github.com/code-423n4/2022-05-alchemix-findings/issues/99) 

# Lines of code

https://github.com/code-423n4/2022-05-alchemix/blob/de65c34c7b6e4e94662bf508e214dcbf327984f4/contracts-full/adapters/yearn/YearnTokenAdapter.sol#L30-L32


# Vulnerability details

Some tokens do not allow for approval of positive amount when allowance is positive already (to handle approval race condition, most known example is USDT).

This can cause the function to stuck whenever a combination of such a token and leftover approval be met. The latter can be possible if, for example, yearn vault is becoming full on a particular wrap() call and accepts only a part of amount, not utilizing the approval fully.

Then each next safeApprove will revert and wrap becomes permanently unavailable. Setting the severity to medium as depositing (wrapping) is core functionality for the contract and its availability is affected.

## Proof of Concept

wrap use one step approve:

https://github.com/code-423n4/2022-05-alchemix/blob/de65c34c7b6e4e94662bf508e214dcbf327984f4/contracts-full/adapters/yearn/YearnTokenAdapter.sol#L30-L32

```solidity
    function wrap(uint256 amount, address recipient) external override returns (uint256) {
        TokenUtils.safeTransferFrom(underlyingToken, msg.sender, address(this), amount);
        TokenUtils.safeApprove(underlyingToken, token, amount);
```

Some ERC20 forbid the approval of positive amount when the allowance is positive:

https://github.com/d-xo/weird-erc20#approval-race-protections

For example, USDT is supported by Yearn and can be the underlying asset:

https://yearn.finance/#/vault/0x7Da96a3891Add058AdA2E826306D812C638D87a7

## Recommended Mitigation Steps

As the most general approach consider approving zero before doing so for the amount:

```solidity
    function wrap(uint256 amount, address recipient) external override returns (uint256) {
        TokenUtils.safeTransferFrom(underlyingToken, msg.sender, address(this), amount);
+      TokenUtils.safeApprove(underlyingToken, token, 0);
        TokenUtils.safeApprove(underlyingToken, token, amount);
```

