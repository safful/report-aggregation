## Tags

- bug
- 0 (Non-critical)
- disagree with severity
- sponsor confirmed

# [Possible Re-entrancy](https://github.com/code-423n4/2021-12-amun-findings/issues/116) 

# Handle

defsec


# Vulnerability details

## Impact

In the contract there is no re-entrancy mitigations. Contracts interact with various outside sources (tokens, aave pools, other possible strategies that may be added in the future, etc). so, for instance, now you have to be careful and do not allow tokens that have a receiver callback (e.g. erc777) or untrustable sources of yield (strategies).


## Proof of Concept

1. Navigate to the following contract functions. (https://github.com/code-423n4/2021-12-amun/blob/cf890dedf2e43ec787e8e5df65726316fda134a1/contracts/basket/contracts/singleJoinExit/EthSingleTokenJoin.sol#L26)

```

        // ######## Wrap TOKEN #########
        address(INTERMEDIATE_TOKEN).call{value: msg.value}("");

        _joinTokenSingle(_joinTokenStruct);

        uint256 remainingIntermediateBalance = INTERMEDIATE_TOKEN.balanceOf(
            address(this)
        );
        if (remainingIntermediateBalance > 0) {
            IWrappedNativeToken(address(INTERMEDIATE_TOKEN)).withdraw(
                remainingIntermediateBalance
            );
            msg.sender.transfer(remainingIntermediateBalance);
        }

```

2. The contract does not follow Check Effect Interaction Pattern. It is vulnerable to re-entrancy. 


3. Locations

```
https://github.com/code-423n4/2021-12-amun/blob/cf890dedf2e43ec787e8e5df65726316fda134a1/contracts/basket/contracts/singleJoinExit/EthSingleTokenJoin.sol#L26

https://github.com/code-423n4/2021-12-amun/blob/cf890dedf2e43ec787e8e5df65726316fda134a1/contracts/basket/contracts/singleJoinExit/EthSingleTokenJoinV2.sol#L26

```


## Tools Used

None

## Recommended Mitigation Steps

Consider using ReentrancyGuard on main action functions: https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/security/ReentrancyGuard.sol

