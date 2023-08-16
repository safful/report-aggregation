## Tags

- bug
- 3 (High Risk)
- resolved
- sponsor confirmed

# [ [WP-H0] Wrong implementation of `EIP712MetaTransaction`](https://github.com/code-423n4/2022-03-rolla-findings/issues/43) 

# Lines of code

https://github.com/code-423n4/2022-03-rolla/blob/efe4a3c1af8d77c5dfb5ba110c3507e67a061bdd/quant-protocol/contracts/utils/EIP712MetaTransaction.sol#L102-L114


# Vulnerability details

1. `EIP712MetaTransaction` is a utils contract that intended to be inherited by concrete (actual) contracts, therefore. it's initializer function should not use the `initializer` modifier, instead, it should use `onlyInitializing` modifier. See the implementation of [openzeppelin `EIP712Upgradeable` initializer function](https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/blob/v4.5.1/contracts/utils/cryptography/draft-EIP712Upgradeable.sol#L48-L57).

https://github.com/code-423n4/2022-03-rolla/blob/efe4a3c1af8d77c5dfb5ba110c3507e67a061bdd/quant-protocol/contracts/utils/EIP712MetaTransaction.sol#L102-L114

```solidity
    /// @notice initialize method for EIP712Upgradeable
    /// @dev called once after initial deployment and every upgrade.
    /// @param _name the user readable name of the signing domain for EIP712
    /// @param _version the current major version of the signing domain for EIP712
    function initializeEIP712(string memory _name, string memory _version)
        public
        initializer
    {
        name = _name;
        version = _version;

        __EIP712_init(_name, _version);
    }
```

Otherwise, when the concrete contract's initializer function (with a `initializer` modifier) is calling EIP712MetaTransaction's initializer function, it will be mistok as reentered and so that it will be reverted (unless in the context of a constructor, e.g. Using @openzeppelin/hardhat-upgrades `deployProxy()` to initialize).

https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/blob/v4.5.1/contracts/proxy/utils/Initializable.sol#L50-L53

```solidity
    /**
     * @dev Modifier to protect an initializer function from being invoked twice.
     */
    modifier initializer() {
        // If the contract is initializing we ignore whether _initialized is set in order to support multiple
        // inheritance patterns, but we only do this in the context of a constructor, because in other contexts the
        // contract may have been reentered.
        require(_initializing ? _isConstructor() : !_initialized, "Initializable: contract is already initialized");

        bool isTopLevelCall = !_initializing;
        if (isTopLevelCall) {
            _initializing = true;
            _initialized = true;
        }

        _;

        if (isTopLevelCall) {
            _initializing = false;
        }
    }
```

See also: https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/releases/tag/v4.4.1


2. `initializer` can only be called once, it can not be "called once after every upgrade".

https://github.com/code-423n4/2022-03-rolla/blob/efe4a3c1af8d77c5dfb5ba110c3507e67a061bdd/quant-protocol/contracts/utils/EIP712MetaTransaction.sol#L102-L114

```solidity
    /// @notice initialize method for EIP712Upgradeable
    /// @dev called once after initial deployment and every upgrade.
    /// @param _name the user readable name of the signing domain for EIP712
    /// @param _version the current major version of the signing domain for EIP712
    function initializeEIP712(string memory _name, string memory _version)
        public
        initializer
    {
        name = _name;
        version = _version;

        __EIP712_init(_name, _version);
    }
```


3. A utils contract that is not expected to be deployed as a standalone contract should be declared as `abstract`. It's `initializer` function should be `internal`.

See the implementation of [openzeppelin `EIP712Upgradeable`](https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/blob/v4.5.1/contracts/utils/cryptography/draft-EIP712Upgradeable.sol#L28).

https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/blob/v4.5.1/contracts/utils/cryptography/draft-EIP712Upgradeable.sol#L28

```solidity
abstract contract EIP712Upgradeable is Initializable {
    // ...
}
```

### Recommendation

Change to:

```solidity
abstract contract EIP712MetaTransaction is EIP712Upgradeable {
    // ...
}
```

```solidity
    /// @notice initialize method for EIP712Upgradeable
    /// @dev called once after initial deployment.
    /// @param _name the user readable name of the signing domain for EIP712
    /// @param _version the current major version of the signing domain for EIP712
    function __EIP712MetaTransaction_init(string memory _name, string memory _version)
        internal
        onlyInitializing
    {
        name = _name;
        version = _version;

        __EIP712_init(_name, _version);
    }
```



