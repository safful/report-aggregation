## Tags

- bug
- 3 (High Risk)
- high quality report
- primary issue
- selected for report
- sponsor confirmed
- edited-by-warden
- H-02

# [It is impossible to slash queued withdrawals that contain a malicious strategy due to a misplacement of the ++i increment](https://github.com/code-423n4/2023-04-eigenlayer-findings/issues/205) 

# Lines of code

https://github.com/code-423n4/2023-04-eigenlayer/blob/main/src/contracts/core/StrategyManager.sol#L556-L579


# Vulnerability details

`StrategyManager::slashQueuedWithdrawal()` contains an `indicesToSkip` parameter to skip malicious strategies, as documented in the [function definition](https://github.com/code-423n4/2023-04-eigenlayer/blob/main/src/contracts/core/StrategyManager.sol#L532-L534):

> so that, e.g., if the slashed QueuedWithdrawal contains a malicious strategy in the `strategies` array which always reverts on calls to its 'withdraw' function, then the malicious strategy can be skipped (with the shares in effect "burned"), while the non-malicious strategies are still called as normal.

The problem is that the function does not work as expected, and `indicesToSkip` is in fact ignored. If the queued withdrawal contains a malicious strategy, it will make the slash always revert.

## Impact

Owners won't be able to slash queued withdrawals that contain a malicious strategy.

An adversary can take advantage of this, and create withdrawal queues that won't be able to be slashed, completely defeating the slash system. The adversary can later complete the withdrawal.

## Proof of Concept

The `++i;` statement in `StrategyManager::slashQueuedWithdrawal()` is misplaced. It is only executed on the `else` statement:

```solidity
    // keeps track of the index in the `indicesToSkip` array
    uint256 indicesToSkipIndex = 0;

    uint256 strategiesLength = queuedWithdrawal.strategies.length;
    for (uint256 i = 0; i < strategiesLength;) {
        // check if the index i matches one of the indices specified in the `indicesToSkip` array
        if (indicesToSkipIndex < indicesToSkip.length && indicesToSkip[indicesToSkipIndex] == i) {
            unchecked {
                ++indicesToSkipIndex;
            }
        } else {
            if (queuedWithdrawal.strategies[i] == beaconChainETHStrategy){
                    //withdraw the beaconChainETH to the recipient
                _withdrawBeaconChainETH(queuedWithdrawal.depositor, recipient, queuedWithdrawal.shares[i]);
            } else {
                // tell the strategy to send the appropriate amount of funds to the recipient
                queuedWithdrawal.strategies[i].withdraw(recipient, tokens[i], queuedWithdrawal.shares[i]);
            }
            unchecked {
                ++i; // @audit
            }
        }
    }
```

[Link to code](https://github.com/code-423n4/2023-04-eigenlayer/blob/main/src/contracts/core/StrategyManager.sol#L556-L579)

Let's suppose that the owner tries to slash a queued withdrawal, and wants to skip the first strategy (index `0`) because it is malicious and makes the whole transaction to revert.

1 . It defines `indicesToSkipIndex = 0`
2 . It enters the `for` loop starting at `i = 0`
3 . `if (indicesToSkipIndex < indicesToSkip.length && indicesToSkip[indicesToSkipIndex] == i)` will be true: `0 < 1 && 0 == 0`
4 . It increments `++indicesToSkipIndex;` to "skip" the malicious strategy, so `indicesToSkipIndex = 1` now.
5 . It goes back to the `for` loop. But `i` hasn't been modified, so `i = 0` still
6 . `if (indicesToSkipIndex < indicesToSkip.length && indicesToSkip[indicesToSkipIndex] == i)` will be false now: `1 < 1 && 0 == 0`
7 . It will enter the `else` statement and attempt to slash the strategy anyway
8 . If the strategy is malicious it will revert, making it impossible to slash
9 . The adversary can later complete the withdrawal

## POC Test

This test shows how the `indicesToSkip` parameter is completely ignored.

For the sake of simplicity of the test, it uses a normal strategy, which will be slashed, proving that it ignores the `indicesToSkip` parameter and it indeed calls `queuedWithdrawal.strategies[i].withdraw()`.

A malicious strategy that makes `withdraw()` revert would make the whole transaction revert (not shown on this test but easily checkeable as the [function won't catch it](https://github.com/code-423n4/2023-04-eigenlayer/blob/main/src/contracts/core/StrategyManager.sol#L536-L579)).

Add this test to `src/tests/StrategyManagerUnit.t.sol` and run `forge test -m "testSlashQueuedWithdrawal_IgnoresIndicesToSkip"`.

```solidity
    function testSlashQueuedWithdrawal_IgnoresIndicesToSkip() external {
        address recipient = address(this);
        uint256 depositAmount = 1e18;
        uint256 withdrawalAmount = depositAmount;
        bool undelegateIfPossible = false;

        // Deposit into strategy and queue a withdrawal
        (IStrategyManager.QueuedWithdrawal memory queuedWithdrawal,,) =
            testQueueWithdrawal_ToSelf_NotBeaconChainETH(depositAmount, withdrawalAmount, undelegateIfPossible);

        // Slash the delegatedOperator
        slasherMock.freezeOperator(queuedWithdrawal.delegatedAddress);

        // Keep track of the balance before the slash attempt
        uint256 balanceBefore = dummyToken.balanceOf(address(recipient));

        // Assert that the strategies array only has one element
        assertEq(queuedWithdrawal.strategies.length, 1);

        // Set `indicesToSkip` so that it should ignore the only strategy
        // As it's the only element, its index is `0`
        uint256[] memory indicesToSkip = new uint256[](1);
        indicesToSkip[0] = 0;

        // Call `slashQueuedWithdrawal()`
        // This should not try to slash the only strategy the queue has, because of the defined `indicesToSkip`
        // But in fact it ignores `indicesToSkip` and attempts to do it anyway
        cheats.startPrank(strategyManager.owner());
        strategyManager.slashQueuedWithdrawal(recipient, queuedWithdrawal, _arrayWithJustDummyToken(), indicesToSkip);
        cheats.stopPrank();

        uint256 balanceAfter = dummyToken.balanceOf(address(recipient));

        // The `indicesToSkip` was completely ignored, and the function attempted the slash anyway
        // It can be asserted due to the fact that it increased the balance
        require(balanceAfter == balanceBefore + withdrawalAmount, "balanceAfter != balanceBefore + withdrawalAmount");
    }
```

## Tools Used

Manual Review

## Recommended Mitigation Steps

Place the `++i` outside of the if/else statement. This way it will increment each time the loop runs.

```diff
    for (uint256 i = 0; i < strategiesLength;) {
        // check if the index i matches one of the indices specified in the `indicesToSkip` array
        if (indicesToSkipIndex < indicesToSkip.length && indicesToSkip[indicesToSkipIndex] == i) {
            unchecked {
                ++indicesToSkipIndex;
            }
        } else {
            if (queuedWithdrawal.strategies[i] == beaconChainETHStrategy){
                    //withdraw the beaconChainETH to the recipient
                _withdrawBeaconChainETH(queuedWithdrawal.depositor, recipient, queuedWithdrawal.shares[i]);
            } else {
                // tell the strategy to send the appropriate amount of funds to the recipient
                queuedWithdrawal.strategies[i].withdraw(recipient, tokens[i], queuedWithdrawal.shares[i]);
            }
-           unchecked {
-               ++i;
-           }
        }
+       unchecked {
+           ++i;
+       }
    }
```





## Assessed type

Loop