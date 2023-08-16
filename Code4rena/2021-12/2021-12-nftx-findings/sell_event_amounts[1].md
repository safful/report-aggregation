## Tags

- bug
- 0 (Non-critical)
- resolved
- sponsor confirmed

# [Sell event amounts[1]](https://github.com/code-423n4/2021-12-nftx-findings/issues/173) 

# Handle

pauliax


# Vulnerability details

## Impact
When emitting Sell event, it assumes that the path is always of length 2, as amounts[1] is used for the ethReceived parameter. However, the path does not have any restrictions on its length, so it is completely possible, that this is not the final amount. Events are used to inform the outside world and this may trick the consumers.

## Recommended Mitigation Steps
amounts[1] should be replaced with amounts[amounts.length - 1]

