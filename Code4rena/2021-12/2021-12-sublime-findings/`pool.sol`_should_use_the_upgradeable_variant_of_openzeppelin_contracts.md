## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [`Pool.sol` should use the Upgradeable variant of OpenZeppelin Contracts](https://github.com/code-423n4/2021-12-sublime-findings/issues/108) 

# Handle

WatchPug


# Vulnerability details

Given that `Pool` is deployed as a proxied contract, it should use the Upgradeable variant of OpenZeppelin Contracts.

https://github.com/code-423n4/2021-12-sublime/blob/9df1b7c4247f8631647c7627a8da9bdc16db8b11/contracts/Pool/PoolFactory.sol#L320-L355

```solidity=320{348}
function _createPool(
    uint256 _poolSize,
    uint256 _borrowRate,
    address _borrowToken,
    address _collateralToken,
    uint256 _idealCollateralRatio,
    uint256 _repaymentInterval,
    uint256 _noOfRepaymentIntervals,
    address _poolSavingsStrategy,
    uint256 _collateralAmount,
    bool _transferFromSavingsAccount,
    bytes32 _salt,
    address _lenderVerifier
) internal {
    bytes memory data = _encodePoolInitCall(
        _poolSize,
        _borrowRate,
        _borrowToken,
        _collateralToken,
        _idealCollateralRatio,
        _repaymentInterval,
        _noOfRepaymentIntervals,
        _poolSavingsStrategy,
        _collateralAmount,
        _transferFromSavingsAccount,
        _lenderVerifier
    );
    bytes32 salt = keccak256(abi.encodePacked(_salt, msg.sender));
    bytes memory bytecode = abi.encodePacked(type(SublimeProxy).creationCode, abi.encode(poolImpl, address(0x01), data));
    uint256 amount = _collateralToken == address(0) ? _collateralAmount : 0;

    address pool = _deploy(amount, salt, bytecode);

    poolRegistry[pool] = true;
    emit PoolCreated(pool, msg.sender);
}
```

Otherwise, the constructor functions of `Pool`'s parent contracts which may change storage at deploy time, won't work for deployed instances.

The effect may be different for different OpenZeppelin libraries.

Take `ReentrancyGuard` for example, the code inside `ReentrancyGuard.sol#constructor` won't work, should use `ReentrancyGuardUpgradeable.sol` instead:

https://github.com/code-423n4/2021-12-sublime/blob/9df1b7c4247f8631647c7627a8da9bdc16db8b11/contracts/Pool/Pool.sol#L6-L8

```solidity=6{6}
import '@openzeppelin/contracts/utils/ReentrancyGuard.sol';
import '@openzeppelin/contracts-upgradeable/proxy/Initializable.sol';
import '@openzeppelin/contracts-upgradeable/token/ERC20/ERC20PausableUpgradeable.sol';
```

https://github.com/code-423n4/2021-12-sublime/blob/9df1b7c4247f8631647c7627a8da9bdc16db8b11/contracts/Pool/Pool.sol#L24-L24

```solidity=24
contract Pool is Initializable, ERC20PausableUpgradeable, IPool, ReentrancyGuard {
```

### Recommendation

Change to:

```solidity=6{6}
import "@openzeppelin/contracts-upgradeable/utils/ReentrancyGuardUpgradeable.sol";
import '@openzeppelin/contracts-upgradeable/proxy/Initializable.sol';
import '@openzeppelin/contracts-upgradeable/token/ERC20/ERC20PausableUpgradeable.sol';
```

```solidity=24
contract Pool is Initializable, ReentrancyGuardUpgradeable, ERC20PausableUpgradeable, IPool {
```

```solidity=133{164}
function initialize(
    uint256 _borrowAmountRequested,
    uint256 _borrowRate,
    address _borrower,
    address _borrowAsset,
    address _collateralAsset,
    uint256 _idealCollateralRatio,
    uint256 _repaymentInterval,
    uint256 _noOfRepaymentIntervals,
    address _poolSavingsStrategy,
    uint256 _collateralAmount,
    bool _transferFromSavingsAccount,
    address _lenderVerifier,
    uint256 _loanWithdrawalDuration,
    uint256 _collectionPeriod
) external payable initializer {
    poolFactory = msg.sender;
    poolConstants.borrowAsset = _borrowAsset;
    poolConstants.idealCollateralRatio = _idealCollateralRatio;
    poolConstants.collateralAsset = _collateralAsset;
    poolConstants.poolSavingsStrategy = _poolSavingsStrategy;
    poolConstants.borrowAmountRequested = _borrowAmountRequested;
    _initialDeposit(_borrower, _collateralAmount, _transferFromSavingsAccount);
    poolConstants.borrower = _borrower;
    poolConstants.borrowRate = _borrowRate;
    poolConstants.noOfRepaymentIntervals = _noOfRepaymentIntervals;
    poolConstants.repaymentInterval = _repaymentInterval;
    poolConstants.lenderVerifier = _lenderVerifier;

    poolConstants.loanStartTime = block.timestamp.add(_collectionPeriod);
    poolConstants.loanWithdrawalDeadline = block.timestamp.add(_collectionPeriod).add(_loanWithdrawalDuration);
    __ReentrancyGuard_init();
    __ERC20_init('Pool Tokens', 'PT');
    try ERC20Upgradeable(_borrowAsset).decimals() returns(uint8 _decimals) {
        _setupDecimals(_decimals);
    } catch(bytes memory) {}
}
```

