## Tags

- bug
- 2 (Med Risk)
- disagree with severity
- sponsor confirmed

# [Change of operators possible from old operators](https://github.com/code-423n4/2022-07-axelar-findings/issues/19) 

# Lines of code

https://github.com/code-423n4/2022-07-axelar/blob/3373c48a71c07cfce856b53afc02ef4fc2357f8c/contracts/AxelarGateway.sol#L268
https://github.com/code-423n4/2022-07-axelar/blob/3373c48a71c07cfce856b53afc02ef4fc2357f8c/contracts/AxelarGateway.sol#L311


# Vulnerability details

## Impact
According to the specifications, only the current operators should be able to transfer operatorship. However, there is one way to circumvent this. Because currentOperators is not updated in the loop, when multiple `transferOperatorship` commands are submitted in the same `execute` call, all will succeed. After the first one, the operators that signed these commands are no longer the current operators, but the call will still succeed.

This also means that one set of operators could submit so many `transferOperatorship` commands in one `execute` call that `OLD_KEY_RETENTION` is reached for all other ones, meaning they would control complete set of currently valid operators.

## Recommended Mitigation Steps
Set `currentOperators` to `false` when the operators were changed.