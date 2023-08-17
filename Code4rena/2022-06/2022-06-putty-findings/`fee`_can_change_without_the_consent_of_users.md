## Tags

- bug
- 2 (Med Risk)
- resolved
- sponsor confirmed

# [`fee` can change without the consent of users](https://github.com/code-423n4/2022-06-putty-findings/issues/422) 

# Lines of code

https://github.com/code-423n4/2022-06-putty/blob/3b6b844bc39e897bd0bbb69897f2deff12dc3893/contracts/src/PuttyV2.sol#L240
 https://github.com/code-423n4/2022-06-putty/blob/3b6b844bc39e897bd0bbb69897f2deff12dc3893/contracts/src/PuttyV2.sol#L497


# Vulnerability details

## Impact
Fees are applied during `withdraw`, but can change between the time the order is filled and its terms are agreed upon and the withdrawal time, leading to a loss of the expected funds for the concerned users.

## Proof of Concept
The scenario would be:

 - Alice and Bob agrees to fill an order at a time fees are 0.1%
 - During the duration of the option, fees are increased to 3%
 - At withdrawal they'll pay 3% of the strike, although they wouldn't have created the order in the first place with such fees


## Recommended Mitigation Steps
Mitigation could be:
 - Store the fees in `Order` and verify that they are correct when the order is filled, so they are hardcoded in the struct
 - Add a timestamp: this wouldn't fully mitigate but would still be better than the current setup
 - Keep past fees and fee change timestamps in memory (for example in an array) to be able to retrieve the creation time fees at withdrawal

