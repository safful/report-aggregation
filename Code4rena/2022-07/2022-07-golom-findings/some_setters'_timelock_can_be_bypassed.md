## Tags

- bug
- 2 (Med Risk)
- resolved
- sponsor confirmed
- old-submission-method
- selected-for-report

# [Some setters' timelock can be bypassed](https://github.com/code-423n4/2022-07-golom-findings/issues/698) 

# Lines of code

https://github.com/code-423n4/2022-07-golom/blob/7bbb55fca61e6bae29e57133c1e45806cbb17aa4/contracts/governance/GolomToken.sol#L58-L72
https://github.com/code-423n4/2022-07-golom/blob/7bbb55fca61e6bae29e57133c1e45806cbb17aa4/contracts/core/GolomTrader.sol#L444-L457
https://github.com/code-423n4/2022-07-golom/blob/7bbb55fca61e6bae29e57133c1e45806cbb17aa4/contracts/rewards/RewardDistributor.sol#L298-L311


# Vulnerability details

## Impact

MED - Function could be impacted

As the timelock does not work as supposed to work, the owner of the contract can bypass timelock.

- effected Functions:
  - `GolomToken`: `setMinter`, `executeSetMinter`
  - `GolomTrader`: `setDistributor`, `executeSetDistributor`
  - `RewardDistributor`: `addVoteEscrow`, `executeAddVoteEscrow`


## Proof of Concept

- [GolomTrader::it can bypass timelock poc](https://gist.github.com/zzzitron/6f950a268d179218cadef74d7acdeeb4#file-2022-07-pocgolomtrader-specs-ts-diff-L15-L26)
- [GolomToken::setMinter it should set the minter with timelock poc](https://gist.github.com/zzzitron/6f950a268d179218cadef74d7acdeeb4#file-2022-07-pocgolomtoken-specs-ts-diff-L10-L29)


The [first poc](https://gist.github.com/zzzitron/6f950a268d179218cadef74d7acdeeb4#file-2022-07-pocgolomtrader-specs-ts-diff-L15-L26
) shows to bypass timelock for `GolomTrader::setDistributor`. The same logic applies for the `RewardDistributor::addVoteEscrow`.
0. The `setDistributor` was called once in the beforeEach block to set the initial distributor. For this exploit to work, the `setDistributor` should be called only once. If `setDistributor` was called more than once, one can set the distributor to zero address (with timelock like in the `GolomToken` case, then set to a new distributor after that)
1. reset distributor to zero address without timelock by calling `executeSetDistributor`
2. set a new distributor without timelock by calling `setDistributor`
3. Rinse and repeat: as long as `setDistributor` is not called multiple times in row, the owner can keep setting distributor without timelock.


A little bit different variation of timelock bypass was found in the `GolomToken`. Although the owner should wait for the timelock to set the minter to zero address, but after that, the owner can set to the new minter without waiting for a timelock. Since the meaning of timelock is to let people know the new minter's implementation, if the owner can bypass that, the timelock is almost meaningless.
The exploitation steps: [the second proof of concept](https://gist.github.com/zzzitron/6f950a268d179218cadef74d7acdeeb4#file-2022-07-pocgolomtoken-specs-ts-diff-L10-L29
)
1. call `setMineter` with zero address
2. wait for the timelock
3. call `executeSetMineter` to set the minter to zero address
4. now the onwer can call `setMineter` with any address and call `executeSetMinter` without waiting for the timelock


The owner can call `executeSetdistributor` even though there is no `pendingDistributor` set before. Also, `setDistributor` sets the new distributor without timelock when the existing distributor's address is zero.

```solidity
// GolomTrader
// almost identical logic was used in `RewardDistributor` to addVoteEscrow
// similar logic was used in `GolomToken` to `setMineter`


444     function setDistributor(address _distributor) external onlyOwner {
445         if (address(distributor) == address(0)) {
446             distributor = Distributor(_distributor);
447         } else {
448             pendingDistributor = _distributor;
449             distributorEnableDate = block.timestamp + 1 days;
450         }
451     }
452
453     /// @notice Executes the set distributor function after the timelock
454     function executeSetDistributor() external onlyOwner {
455         require(distributorEnableDate <= block.timestamp, 'not allowed');
456         distributor = Distributor(pendingDistributor);
457     }
```


## Tools Used

None

## Recommended Mitigation Steps

To mitigate, execute functions can check whether pendingDistributor is not zero. It will ensure that the setters are called before executing them, as well as prevent to set to zero addresses.

<!-- zzzitron 01M -->



