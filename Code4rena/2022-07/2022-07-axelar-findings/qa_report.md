## Tags

- bug
- QA (Quality Assurance)
- sponsor confirmed

# [QA Report](https://github.com/code-423n4/2022-07-axelar-findings/issues/73) 

## [NAZ-L1] `receive()` Function Should Emit An Event
**Severity**: Low
**Context**: [`DepositReceiver.sol#L29`](https://github.com/code-423n4/2022-07-axelar/blob/main/contracts/deposit-service/DepositReceiver.sol#L29), [`AxelarDepositServiceProxy.sol#L13`](https://github.com/code-423n4/2022-07-axelar/blob/main/contracts/deposit-service/AxelarDepositServiceProxy.sol#L13)

**Description**:
Consider emitting an event inside this function with `msg.sender` and `msg.value` as the parameters. This would make it easier to track incoming ether transfers.

**Recommendation**:
Add events to the `receive()` functions. 


## [NAZ-L2] Local Variable Shadowing
**Severity**: Low
**Context**: [`AxelarDepositService.sol#L19 (both gateway && wrappedSymbol)`](https://github.com/code-423n4/2022-07-axelar/blob/main/contracts/deposit-service/AxelarDepositService.sol#L19)

**Description**:
These Variables shadow state variables. As a result, the use of them locally might be incorrect.

**Recommendation**:
Rename the local variables that shadow another component.


## [NAZ-N1] Code Contains Empty Blocks
**Severity** Informational
**Context**: [`AxelarDepositServiceProxy.sol#L13`](https://github.com/code-423n4/2022-07-axelar/blob/main/contracts/deposit-service/AxelarDepositServiceProxy.sol#L13), [`AxelarAuthWeighted.sol#L101`](https://github.com/code-423n4/2022-07-axelar/blob/main/contracts/auth/AxelarAuthWeighted.sol#L101)

**Description**:
It's best practice that when there is an empty block, to add a comment in the block exmplaining why it's empty.

**Recommendation**:
Consider adding `/* Comment on why */` to the empty block.


## [NAZ-N2] Variable Naming Convention
**Severity** Informational
**Context**: [`AxelarGateway.sol#L45-L46`](https://github.com/code-423n4/2022-07-axelar/blob/main/contracts/AxelarGateway.sol#L45-L46)

**Description**:
The linked variables do not conform to the standard naming convention of Solidity whereby functions and variable names(local, state and immutable) utilize the `mixedCase` format unless variables are declared as `constant` in which case they utilize the `UPPER_CASE_WITH_UNDERSCORES` format.

**Recommendation**:
Naming conventions utilized by the linked statements are adjusted to reflect the correcttype of declaration according to the Solidity style guide.


## [NAZ-N3] Missing Use of `solhint-disable-next-line`
**Severity**: Informational
**Context**:[`AxelarGateway.sol#L157`](https://github.com/code-423n4/2022-07-axelar/blob/main/contracts/AxelarGateway.sol#L157), [`AxelarGateway.sol#L229`](https://github.com/code-423n4/2022-07-axelar/blob/main/contracts/AxelarGateway.sol#L229), [`AxelarGateway.sol#L320`](https://github.com/code-423n4/2022-07-axelar/blob/main/contracts/AxelarGateway.sol#L320), [`AxelarGateway.sol#L344`](https://github.com/code-423n4/2022-07-axelar/blob/main/contracts/AxelarGateway.sol#L344), [`AxelarGateway.sol#L461`](https://github.com/code-423n4/2022-07-axelar/blob/main/contracts/AxelarGateway.sol#L461), [`AxelarGateway.sol#L615`](https://github.com/code-423n4/2022-07-axelar/blob/main/contracts/AxelarGateway.sol#L615), [`AxelarAuthWeighted.sol#L101`](https://github.com/code-423n4/2022-07-axelar/blob/main/contracts/auth/AxelarAuthWeighted.sol#L101)

**Description**:
`solhint-disable-next-line` is used elsewhere for small linter errors and can be used here to disable these errors.

**Recommendation**:
Consider adding `solhint-disable-next-line`.


## [NAZ-N4] Commented Out Code
**Severity**: Informational
**Context**: [`AxelarGateway.sol#L22-L24`](https://github.com/code-423n4/2022-07-axelar/blob/main/contracts/AxelarGateway.sol#L22-L24)

**Description**:
There is commented code that makes the code messy and unneeded. 

**Recommendation**:
Remove the commented out code.


## [NAZ-N5] Floating Pragma
**Severity**: Informational
**Context**: [`IAxelarGasService.sol`](https://github.com/code-423n4/2022-07-axelar/blob/main/contracts/interfaces/IAxelarGasService.sol), [`IAxelarDepositService.sol`](https://github.com/code-423n4/2022-07-axelar/blob/main/contracts/interfaces/IAxelarDepositService.sol), [`IAxelarAuthWeighted.sol`](https://github.com/code-423n4/2022-07-axelar/blob/main/contracts/interfaces/IAxelarAuthWeighted.sol), [`IDepositBase.sol`](https://github.com/code-423n4/2022-07-axelar/blob/main/contracts/interfaces/IDepositBase.sol)

**Description**:
Contracts should be deployed with the same compiler version and flags that they have been tested with thoroughly. Locking the pragma helps to ensure that contracts do not accidentally get deployed using, for example, an outdated compiler version that might introduce bugs that affect the contract system negatively.

**Recommendation**: 
Lock the pragma version.


## [NAZ-N6] Missing or Incomplete NatSpec
**Severity**: Informational
**Context**: [`All Contracts`](https://github.com/code-423n4/2022-07-axelar)

**Description**:
Some functions are missing @notice/@dev NatSpec comments for the function, @param for all/some of their parameters and @return for return values. Given that NatSpec is an important part of code documentation, this affects code comprehension, auditability and usability.

**Recommendation**:
Add in full NatSpec comments for all functions to have complete code documentation for future use.