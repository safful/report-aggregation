## Tags

- bug
- 2 (Med Risk)
- satisfactory
- selected for report
- sponsor confirmed
- edited-by-warden
- M-18

# [If name is changed then the domain seperator would be wrong.](https://github.com/code-423n4/2023-01-reserve-findings/issues/211) 

# Lines of code

https://github.com/reserve-protocol/protocol/blob/df7ecadc2bae74244ace5e8b39e94bc992903158/contracts/p1/StRSR.sol#L803
https://github.com/reserve-protocol/protocol/blob/df7ecadc2bae74244ace5e8b39e94bc992903158/contracts/p1/StRSR.sol#L791


# Vulnerability details

### Vulnerability Details

In `StRSR.sol` the `_domainSeparatorV4` is calculated using the EIP-721 standard, which uses the `name` and `version` that are passed in the init at the function call `__EIP712_init(name, "1");`

Now, governance can change this `name` anytime using the following function:

```solidity
function setName(string calldata name_) external governance {
        name = name_;
    }
```

After that call the domain seperator would still be calculated using the old name, which shouldn’t be the case. 

### Impact

The permit transactions and vote delegation would be reverted if the domain seperator is wrong. 

### Recommendation

While changing the name in in setName function. update the domain seperator.