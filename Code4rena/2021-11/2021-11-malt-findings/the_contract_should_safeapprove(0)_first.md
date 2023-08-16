## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [The Contract Should safeApprove(0) first](https://github.com/code-423n4/2021-11-malt-findings/issues/41) 

# Handle

defsec


# Vulnerability details

## Impact

Some tokens (like USDT L199) do not work when changing the allowance from an existing non-zero allowance value.
They must first be approved by zero and then the actual allowance must be approved.

```
IERC20(token).safeApprove(address(operator), 0);
IERC20(token).safeApprove(address(operator), amount);
```

## Proof of Concept

1. Navigate to the following contracts.

```

https://github.com/code-423n4/2021-11-malt/blob/d3f6a57ba6694b47389b16d9d0a36a956c5e6a94/src/contracts/DexHandlers/UniswapHandler.sol#L167

https://github.com/code-423n4/2021-11-malt/blob/d3f6a57ba6694b47389b16d9d0a36a956c5e6a94/src/contracts/AuctionParticipant.sol#L59

https://github.com/code-423n4/2021-11-malt/blob/d3f6a57ba6694b47389b16d9d0a36a956c5e6a94/src/contracts/StabilizerNode.sol#L252

https://github.com/code-423n4/2021-11-malt/blob/d3f6a57ba6694b47389b16d9d0a36a956c5e6a94/src/contracts/RewardReinvestor.sol#L107
```
2.  When trying to re-approve an already approved token, all transactions revert and the protocol cannot be used.

## Tools Used

None

## Recommended Mitigation Steps

Approve with a zero amount first before setting the actual amount.

