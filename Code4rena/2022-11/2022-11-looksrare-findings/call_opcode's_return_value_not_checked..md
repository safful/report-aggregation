## Tags

- bug
- 2 (Med Risk)
- downgraded by judge
- primary issue
- satisfactory
- selected for report
- sponsor confirmed
- M-05

# [call opcode's return value not checked.](https://github.com/code-423n4/2022-11-looksrare-findings/issues/241) 

# Lines of code

https://github.com/code-423n4/2022-11-looksrare/blob/main/contracts/lowLevelCallers/LowLevelETH.sol#L35
https://github.com/code-423n4/2022-11-looksrare/blob/main/contracts/lowLevelCallers/LowLevelETH.sol#L46
https://github.com/code-423n4/2022-11-looksrare/blob/main/contracts/lowLevelCallers/LowLevelETH.sol#L57


# Vulnerability details

## Impact
The `call` opcode's return value not checked, which could leads to the `originator` lose funds.

## Proof of Concept
The caller of `LooksRareAggregator.sol::execute` could be a contract who may not implement the `fallback` or `receive` function, when a call to it with value sent, it will revert, thus failed to receive the ETH.

Let's imagine the contract call the `execute` function to buy multiple NFTs with ETH as the payout currency and make the `isAtomic` parameter being false. Since the batch buy of NFTs is not atomic, the failed transactions in LooksRare or Seaport marketplace will return the passed ETH. The contract doesn't implement the `fallback/receive` function and the call opcode's return value not checked, thus the ETH value will be trapped in the `LooksRareAggregator` contract until the next user call the `execute` function and the trapped ETH is returned to him. The `originator` lose funds.

```solidity
    function _returnETHIfAny(address recipient) internal {
        assembly {
            if gt(selfbalance(), 0) {
                let status := call(gas(), recipient, selfbalance(), 0, 0, 0, 0) // @audit-issue status not checked.
            }
        }
    }
```


## Tools Used
Manual review

## Recommended Mitigation Steps
 check the return value the `call` opcode.
```solidity
    function _returnETHIfAny() internal {
        bool status;
        assembly {
            if gt(selfbalance(), 0) {
                status := call(gas(), caller(), selfbalance(), 0, 0, 0, 0) // @audit-issue [MED] status not checked
            }
        }
        if (!status) revert ETHTransferFail();
    }

    function _returnETHIfAny(address recipient) internal {
        bool status;
        assembly {
            if gt(selfbalance(), 0) {
                status := call(gas(), recipient, selfbalance(), 0, 0, 0, 0) // @audit-issue status not checked.
            }
        }
        if (!status) revert ETHTransferFail();
    }
function _returnETHIfAnyWithOneWeiLeft() internal {
        bool status;
        assembly {
            if gt(selfbalance(), 1) {
                status := call(gas(), caller(), sub(selfbalance(), 1), 0, 0, 0, 0)
            }
        }
        if (!status) revert ETHTransferFail();
    }
```

