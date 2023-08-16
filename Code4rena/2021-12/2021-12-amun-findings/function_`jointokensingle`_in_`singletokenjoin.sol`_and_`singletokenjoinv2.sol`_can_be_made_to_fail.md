## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Function `joinTokenSingle` in `SingleTokenJoin.sol` and `SingleTokenJoinV2.sol` can be made to fail](https://github.com/code-423n4/2021-12-amun-findings/issues/81) 

# Handle

pmerkleplant


# Vulnerability details

## Impact

There's a griefing attack vulnerability in the function `joinTokenSingle` in 
`SingleTokenJoin.sol` as well as `SingleTokenJoinV2.sol` which makes any user
transaction fail with "FAILED_OUTPUT_AMOUNT".

### Proof of Concept

The `JoinTokenStruct` argument for `joinTokenSingle` includes a field `outputAmount`
to indicate the amount of tokens the user should receive after joining a basket
(see line [135](https://github.com/code-423n4/2021-12-amun/blob/main/contracts/basket/contracts/singleJoinExit/SingleTokenJoin.sol#L135) and [130](https://github.com/code-423n4/2021-12-amun/blob/main/contracts/basket/contracts/singleJoinExit/SingleTokenJoinV2.sol#L130)).

However, this amount is compared to the contract's balance of the token and
reverts if the amount is unequal.

If an attacker sends some amount of a basket's token to the contract, every call
to this function will fail as long as the output token equals the attacker's token send.

## Recommended Mitigation Steps

Refactor the `require` statement to expect at least the `outputAmount` of tokens,
i.e. `require(outputAmount >= _joinTokenStruct.outputAmount)`.

