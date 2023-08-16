## Tags

- bug
- 1 (Low Risk)
- resolved
- sponsor confirmed

# [`pendingOwner` should be reset to `address(0)` after `acceptOwner()` is successfully called](https://github.com/code-423n4/2022-01-timeswap-findings/issues/83) 

# Handle

Dravee


# Vulnerability details

## Impact
The `acceptOwner()` external function can be called indefinitely instead of only once.
The contract's state doesn't reflect reality.
The code doesn't follow the standard implementation of a 2-step ownership transfer.

## Proof of Concept
Here's the current `acceptOwner()` external function, which lacks a reset of `pendingOwner` to `address(0)` :
```
    function acceptOwner() external override {
        require(msg.sender == pendingOwner, 'E102');
        owner = msg.sender;

        emit AcceptOwner(msg.sender);
    }
```

## Tools Used
VS Code

## Recommended Mitigation Steps
Change the code to:
```
    function acceptOwner() external override {
        require(msg.sender == pendingOwner, 'E102');
        owner = msg.sender;
        pendingOwner = address(0); // @audit : line to add

        emit AcceptOwner(msg.sender);
    }
```

