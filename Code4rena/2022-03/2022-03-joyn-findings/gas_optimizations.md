## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Gas Optimizations](https://github.com/code-423n4/2022-03-joyn-findings/issues/69) 

**[S]**: Suggested optimation, save a decent amount of gas without compromising readability;

**[M]**: Minor optimation, the amount of gas saved is minor, change when you see fit;

**[N]**: Non-preferred, the amount of gas saved is at cost of readability, only apply when gas saving is a top priority.

## [S] Avoid unnecessary external call can save gas

Every call to an external contract costs a decent amount of gas. For optimization of gas usage, avoiding unnecessary external call can save gas if possible.

For example:

https://github.com/code-423n4/2022-03-joyn/blob/c9297ccd925ebb2c44dbc6eaa3effd8db5d2368a/royalty-vault/contracts/RoyaltyVaultFactory.sol#L37-L50

```solidity
    function createVault(address _splitter, address _royaltyAsset)
        external
        returns (address vault)
    {
        splitterProxy = _splitter;
        royaltyAsset = _royaltyAsset;

        vault = address(
            new ProxyVault{salt: keccak256(abi.encode(_splitter))}()
        );

        delete splitterProxy;
        delete royaltyAsset;
    }
```

https://github.com/code-423n4/2022-03-joyn/blob/c9297ccd925ebb2c44dbc6eaa3effd8db5d2368a/royalty-vault/contracts/ProxyVault.sol#L16-L22

```solidity
    constructor() {
        royaltyVault = IVaultFactory(msg.sender).royaltyVault();
        splitterProxy = IVaultFactory(msg.sender).splitterProxy();
        royaltyAsset = IVaultFactory(msg.sender).royaltyAsset();
        platformFee = IVaultFactory(msg.sender).platformFee();
        platformFeeRecipient = IVaultFactory(msg.sender).platformFeeRecipient();
    }
```

Can be changed to:

```solidity
    constructor(address _royaltyVault, address _splitterProxy, address _royaltyAsset, uint256 _platformFee, address _platformFeeRecipient) {
        royaltyVault = _royaltyVault;
        splitterProxy = _splitterProxy;
        royaltyAsset = _royaltyAsset;
        platformFee = _platformFee;
        platformFeeRecipient = _platformFeeRecipient;
    }
```

```solidity
    function createVault(address _splitter, address _royaltyAsset)
        external
        returns (address vault)
    {
        splitterProxy = _splitter;
        royaltyAsset = _royaltyAsset;

        vault = address(
            new ProxyVault{salt: keccak256(abi.encode(_splitter))}(royaltyVault, splitterProxy, royaltyAsset, platformFee, platformFeeRecipient)
        );

        delete splitterProxy;
        delete royaltyAsset;
    }
```

It can save 5 times of external calls.

## [M] Setting `uint256` variables to `0` is redundant

https://github.com/code-423n4/2022-03-joyn/blob/c9297ccd925ebb2c44dbc6eaa3effd8db5d2368a/splits/contracts/Splitter.sol#L49-L49

```solidity
        uint256 amount = 0;
```

Setting `uint256` variables to `0` is redundant as they default to `0`.

Other examples include:

https://github.com/code-423n4/2022-03-joyn/blob/c9297ccd925ebb2c44dbc6eaa3effd8db5d2368a/core-contracts/contracts/CoreCollection.sol#L279-L279

## [S] Cache array length in for loops can save gas

Reading array length at each iteration of the loop takes 6 gas (3 for mload and 3 to place memory_offset) in the stack.

Caching the array length in the stack saves around 3 gas per iteration.

Instances include:

- `CoreFactory.sol#createProject()`

    https://github.com/code-423n4/2022-03-joyn/blob/c9297ccd925ebb2c44dbc6eaa3effd8db5d2368a/core-contracts/contracts/CoreFactory.sol#L79-L92

- `Splitter.sol#verifyProof()`

    https://github.com/code-423n4/2022-03-joyn/blob/c9297ccd925ebb2c44dbc6eaa3effd8db5d2368a/splits/contracts/Splitter.sol#L274-L288

## [M] `++i` is more efficient than `i++`

Using `++i` is more gas efficient than `i++`, especially in for loops.

For example:

https://github.com/code-423n4/2022-03-joyn/blob/c9297ccd925ebb2c44dbc6eaa3effd8db5d2368a/splits/contracts/Splitter.sol#L274-L274

```solidity
        for (uint256 i = 0; i < proof.length; i++)
```

Change to:

```solidity
        for (uint256 i = 0; i < proof.length; +i)
```

## [M] Unused constant variable

Unused constant variables in contracts increase contract size and gas usage at deployment.

 https://github.com/code-423n4/2022-03-joyn/blob/c9297ccd925ebb2c44dbc6eaa3effd8db5d2368a/splits/contracts/Splitter.sol#L14-L15

```solidity
    uint256 public constant PERCENTAGE_SCALE = 10e5;

```

In the contract, the constant variable `PERCENTAGE_SCALE` is set once but has never been read, therefore it can be removed.

## [M] Use short reason strings can save gas

Every reason string takes at least 32 bytes.

Use short reason strings that fits in 32 bytes or it will become more expensive.

Instances include:

https://github.com/code-423n4/2022-03-joyn/blob/c9297ccd925ebb2c44dbc6eaa3effd8db5d2368a/core-contracts/contracts/CoreCollection.sol#L146-L146

```solidity
require(amount > 0, "CoreCollection: Amount should be greater than 0");
```

https://github.com/code-423n4/2022-03-joyn/blob/c9297ccd925ebb2c44dbc6eaa3effd8db5d2368a/core-contracts/contracts/CoreCollection.sol#L189-L192

```solidity
        require(
            msg.sender == splitFactory || msg.sender == owner(),
            "CoreCollection: Only Split Factory or owner can initialize vault."
        );
```

https://github.com/code-423n4/2022-03-joyn/blob/c9297ccd925ebb2c44dbc6eaa3effd8db5d2368a/core-contracts/contracts/CoreCollection.sol#L220-L223

```solidity
        require(
            startingIndex == 0,
            "CoreCollection: Starting index is already set"
        );
```

## [M] Unused events

Unused events increase contract size and gas usage at deployment.

https://github.com/code-423n4/2022-03-joyn/blob/c9297ccd925ebb2c44dbc6eaa3effd8db5d2368a/royalty-vault/contracts/RoyaltyVaultFactory.sol#L19-L19

```solidity
    event VaultCreated(address vault);
```

`VaultCreated` is unused.

## [M] Unused function parameters

Unused private function increase contract size and gas usage at deployment.

https://github.com/code-423n4/2022-03-joyn/blob/c9297ccd925ebb2c44dbc6eaa3effd8db5d2368a/splits/contracts/Splitter.sol#L248-L257

```solidity
    function attemptETHTransfer(address to, uint256 value)
        private
        returns (bool)
    {
        // Here increase the gas limit a reasonable amount above the default, and try
        // to send ETH to the recipient.
        // NOTE: This might allow the recipient to attempt a limited reentrancy attack.
        (bool success, ) = to.call{value: value, gas: 30000}("");
        return success;
    }
```

`attemptETHTransfer()` is unused.

https://github.com/code-423n4/2022-03-joyn/blob/c9297ccd925ebb2c44dbc6eaa3effd8db5d2368a/splits/contracts/Splitter.sol#L217-L224

```solidity
    function amountFromPercent(uint256 amount, uint32 percent)
        private
        pure
        returns (uint256)
    {
        // Solidity 0.8.0 lets us do this without SafeMath.
        return (amount * percent) / 100;
    }
```

`amountFromPercent()` is unused.

## [S] Use `immutable` instead of getter to save gas

https://github.com/code-423n4/2022-03-joyn/blob/c9297ccd925ebb2c44dbc6eaa3effd8db5d2368a/core-contracts/contracts/CoreProxy.sol#L8-L37

```solidity
contract CoreProxy is Ownable {
    address private immutable _implement;

    constructor(address _imp) {
        _implement = _imp;
    }

    fallback() external {
        address _impl = implement();
        assembly {
            let ptr := mload(0x40)
            calldatacopy(ptr, 0, calldatasize())
            let result := delegatecall(gas(), _impl, ptr, calldatasize(), 0, 0)
            let size := returndatasize()
            returndatacopy(ptr, 0, size)

            switch result
            case 0 {
                revert(ptr, size)
            }
            default {
                return(ptr, size)
            }
        }
    }

    function implement() public view returns (address) {
        return _implement;
    }
}
```

### Recommendation

Change to:

```solidity
contract CoreProxy is Ownable {
    address private immutable _implement;

    constructor(address _imp) {
        _implement = _imp;
    }

    fallback() external {
        address _impl = _implement;
        assembly {
            let ptr := mload(0x40)
            calldatacopy(ptr, 0, calldatasize())
            let result := delegatecall(gas(), _impl, ptr, calldatasize(), 0, 0)
            let size := returndatasize()
            returndatacopy(ptr, 0, size)

            switch result
            case 0 {
                revert(ptr, size)
            }
            default {
                return(ptr, size)
            }
        }
    }

    function implement() external view returns (address) {
        return _implement;
    }
}
```
