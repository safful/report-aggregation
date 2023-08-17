## Tags

- bug
- 2 (Med Risk)
- primary issue
- satisfactory
- selected for report
- sponsor confirmed
- edited-by-warden
- M-27

# [rotateNodeRunnerOfSmartWallet is vulnerable to a frontrun attack](https://github.com/code-423n4/2022-11-stakehouse-findings/issues/386) 

# Lines of code

https://github.com/code-423n4/2022-11-stakehouse/blob/main/contracts/liquid-staking/LiquidStakingManager.sol#L356
https://github.com/code-423n4/2022-11-stakehouse/blob/main/contracts/liquid-staking/LiquidStakingManager.sol#L369


# Vulnerability details

## Impact
As the `rotateNodeRunnerOfSmartWallet` function can be called by anyone who is a node runner in the LSD network, this function is vulnerable to a frontrun attack in the case of this node runner being malicious.

## Proof of Concept
If that is the current node runner is malicious, the DAO would purposely call this same `rotateNodeRunnerOfSmartWallet` with the `_wasPreviousNodeRunnerMalicious` flag turned on.
An actual node runner that has been malicious could monitor the mempool and frontrun the DAO transaction that wanted to slash it and submit the transaction before the DAO to avoid getting banned and rotate their EOA representation of the node.

```solidity
if (msg.sender == dao && _wasPreviousNodeRunnerMalicious) {
    bannedNodeRunners[_current] = true;
    emit NodeRunnerBanned(_current);
}
```

When the DAO transaction would go through, it would revert when it's checking if the current (old) node representative is still a wallet, but it's not because the mapping value has been deleted before.

```solidity
address wallet = smartWalletOfNodeRunner[_current];
require(wallet != address(0), "Wallet does not exist");
```

## Tools Used
Manual inspection

## Recommended Mitigation Steps
Restrict this function to DAO only with the `onlyDAO` modifier.

```solidity
// - function rotateNodeRunnerOfSmartWallet(address _current, address _new, bool _wasPreviousNodeRunnerMalicious) external {
+ function rotateNodeRunnerOfSmartWallet(address _current, address _new, bool _wasPreviousNodeRunnerMalicious) onlyDAO external {
    require(_new != address(0) && _current != _new, "New is zero or current");

    address wallet = smartWalletOfNodeRunner[_current];
    require(wallet != address(0), "Wallet does not exist");
    require(_current == msg.sender || dao == msg.sender, "Not current owner or DAO");

    address newRunnerCurrentWallet = smartWalletOfNodeRunner[_new];
    require(newRunnerCurrentWallet == address(0), "New runner has a wallet");

    smartWalletOfNodeRunner[_new] = wallet;
    nodeRunnerOfSmartWallet[wallet] = _new;

    delete smartWalletOfNodeRunner[_current];

    // - if (msg.sender == dao && _wasPreviousNodeRunnerMalicious) {
    if (_wasPreviousNodeRunnerMalicious) {
        bannedNodeRunners[_current] = true;
        emit NodeRunnerBanned(_current);
    }

    emit NodeRunnerOfSmartWalletRotated(wallet, _current, _new);
}