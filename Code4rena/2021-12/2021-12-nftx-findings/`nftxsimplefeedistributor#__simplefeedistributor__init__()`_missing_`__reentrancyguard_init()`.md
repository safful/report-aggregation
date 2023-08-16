## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [`NFTXSimpleFeeDistributor#__SimpleFeeDistributor__init__()` Missing `__ReentrancyGuard_init()`](https://github.com/code-423n4/2021-12-nftx-findings/issues/90) 

# Handle

WatchPug


# Vulnerability details

https://github.com/code-423n4/2021-12-nftx/blob/194073f750b7e2c9a886ece34b6382b4f1355f36/nftx-protocol-v2/contracts/solidity/NFTXSimpleFeeDistributor.sol#L15-L15

```solidity=15
contract NFTXSimpleFeeDistributor is INFTXSimpleFeeDistributor, ReentrancyGuardUpgradeable, PausableUpgradeable {
```

https://github.com/code-423n4/2021-12-nftx/blob/194073f750b7e2c9a886ece34b6382b4f1355f36/nftx-protocol-v2/contracts/solidity/NFTXSimpleFeeDistributor.sol#L41-L47

```solidity=41
  function __SimpleFeeDistributor__init__(address _lpStaking, address _treasury) public override initializer {
    __Pausable_init();
    setTreasuryAddress(_treasury);
    setLPStakingAddress(_lpStaking);

    _addReceiver(0.8 ether, lpStaking, true);
  }
```

For the upgradeable variants of OpenZipplin contracts, they should be initialized by calling the `__***_init()` function in the initializer function.

Therefore, `__SimpleFeeDistributor__init__()` should call `__ReentrancyGuard_init()` at L42.

