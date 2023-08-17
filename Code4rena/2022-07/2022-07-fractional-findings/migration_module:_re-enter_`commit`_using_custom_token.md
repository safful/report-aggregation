## Tags

- bug
- 3 (High Risk)
- sponsor confirmed
- old-submission-method

# [Migration Module: Re-enter `commit` using custom token](https://github.com/code-423n4/2022-07-fractional-findings/issues/576) 

# Lines of code

https://github.com/code-423n4/2022-07-fractional/blob/8f2697ae727c60c93ea47276f8fa128369abfe51/src/modules/Migration.sol#L207-L212


# Vulnerability details

## Impact

HIGH - Assets can be compromised directly.
One can drain eth out from migration module to buyout module using custom made FERC1155 token.

## Proof of Concept

- [proof of concept: `testCommitReenter_poc`](https://gist.github.com/zzzitron/24c02e069b428f7a95ebc6c931e29b4e#file-2022-07-fractionalv2-poc-modules-t-sol-L283-L339)
- [custom made FERC1155 for the attack](https://gist.github.com/zzzitron/24c02e069b428f7a95ebc6c931e29b4e#file-2022-07-fractionalv2-poc-modules-t-sol-L6-L63)

The proof of concept shows a scenario where alice is draining migration module using custom made FERC1155 token.

1. setup: other people are using migration module and they deposited some eth. (using alice and bob just to simplify the set up process)
2. alice prepared the custom FERC1155 (let's say `evil_token`)
3. alice create a vault with the `evil_token`
4. alice proposes and joins with 0.5 ether
5. when alice calls `commit`, the `evil_token` will reenter `commit` and send money to buyout module

Note: For a simplicity, the `evil_token` reenters for a fixed number of times. But one can adjust to drain all the eth in the migration module.
Note2: For now the eth is in the buyout module, but given the current implementation of `buyout` module, the same actor can drain eth from buyout.

The `commit` function is not written in Checks, Effects, Interactions (CEI) patterns. 

```solidity
// modules/Migration.sol::commit
// proposal.isCommited and started are set after the out going calls (i.e. start, setApprovalFor)
// Mitigation idea: set the values before the out going calls

206         if (currentPrice > proposal.targetPrice) {
207             // Sets token approval to the buyout contract
208             IFERC1155(token).setApprovalFor(address(buyout), id, true);
209             // Starts the buyout process
210             IBuyout(buyout).start{value: proposal.totalEth}(_vault);
211             proposal.isCommited = true;
212             started = true;
213         }
```

## Tools Used

foundry

## Recommended Mitigation Steps

Follow Checks, Effects, Interactions patterns. One can also consider adding reentrancy guard.





