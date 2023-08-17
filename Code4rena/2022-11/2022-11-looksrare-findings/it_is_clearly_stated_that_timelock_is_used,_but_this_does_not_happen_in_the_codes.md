## Tags

- bug
- 2 (Med Risk)
- primary issue
- satisfactory
- selected for report
- sponsor confirmed
- M-03

# [It is clearly stated that timelock is used, but this does not happen in the codes](https://github.com/code-423n4/2022-11-looksrare-findings/issues/127) 

# Lines of code

https://github.com/code-423n4/2022-11-looksrare/blob/main/contracts/OwnableTwoSteps.sol#L11


# Vulnerability details

## Impact

It is stated in the documents that "contract ownership management" is used with a timelock;

```js
README.md:
  122: - Does it use a timelock function?: Yes but only for contract ownership management and not business critical functions.
```

However, the `function _setupDelayForRenouncingOwnership` where timelock is specified in the `OwnableTwoSteps.sol` contract where `owner` privileges are set is not used in the project, so a timelock cannot be mentioned.

```solidity
 function _setupDelayForRenouncingOwnership(uint256 _delay) internal {
        delay = _delay;
    }
```


This is stated in the NatSpec comments but there is no definition as stated in the comments;

```solidity
contracts/OwnableTwoSteps.sol:
  40:      *         Delay (for the timelock) must be set by the contract that inherits from this.

```


## Tools Used
Manuel Code Review


## Recommended Mitigation Steps

```diff

contracts/OwnableTwoSteps.sol:

    // Delay for the timelock (in seconds)
    uint256 public delay;

  43       */
  44:     constructor(uint256 _delay) {
  45:         owner = msg.sender;
  +           delay = _delay;
  46:     }
  47: 
  48      /**

  ```

