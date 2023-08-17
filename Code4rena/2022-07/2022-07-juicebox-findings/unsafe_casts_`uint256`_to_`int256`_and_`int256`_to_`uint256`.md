## Tags

- bug
- documentation
- disagree with severity
- QA (Quality Assurance)
- sponsor confirmed
- valid

# [Unsafe casts `uint256` to `int256` and `int256` to `uint256`](https://github.com/code-423n4/2022-07-juicebox-findings/issues/293) 

# Lines of code

https://github.com/jbx-protocol/juice-contracts-v2-code4rena/blob/733810a0339a5c0cb608345e6fc66a6edeac13cc/contracts/JBController.sol#L816
https://github.com/jbx-protocol/juice-contracts-v2-code4rena/blob/733810a0339a5c0cb608345e6fc66a6edeac13cc/contracts/JBController.sol#L668
https://github.com/jbx-protocol/juice-contracts-v2-code4rena/blob/733810a0339a5c0cb608345e6fc66a6edeac13cc/contracts/JBController.sol#L681
https://github.com/jbx-protocol/juice-contracts-v2-code4rena/blob/733810a0339a5c0cb608345e6fc66a6edeac13cc/contracts/JBController.sol#L743
https://github.com/jbx-protocol/juice-contracts-v2-code4rena/blob/733810a0339a5c0cb608345e6fc66a6edeac13cc/contracts/JBController.sol#L785
https://github.com/jbx-protocol/juice-contracts-v2-code4rena/blob/733810a0339a5c0cb608345e6fc66a6edeac13cc/contracts/JBController.sol#L859


# Vulnerability details

### Impact

The JBController contract performs many unsafe casts `uint256` to `int256` and `int256` to `uint256`
In example:
- the cast `-1`(int256) to uint256 was `2**256 - 1`
- the cast `2**255`(uint256) to int256 was `- 2**255`

### Proof of Concept

`int256` to `uint256`:
- [L816: `if (uint256(_processedTokenTrackerOf[_projectId]) != tokenStore.totalSupplyOf(_projectId))`](https://github.com/jbx-protocol/juice-contracts-v2-code4rena/blob/733810a0339a5c0cb608345e6fc66a6edeac13cc/contracts/JBController.sol#L816)

`uint256` to `int256`:
- [L668: `int256(_tokenCount);`](https://github.com/jbx-protocol/juice-contracts-v2-code4rena/blob/733810a0339a5c0cb608345e6fc66a6edeac13cc/contracts/JBController.sol#L668)
- [L681: `int256(beneficiaryTokenCount);`](https://github.com/jbx-protocol/juice-contracts-v2-code4rena/blob/733810a0339a5c0cb608345e6fc66a6edeac13cc/contracts/JBController.sol#L681)
- [L743: `int256(_tokenCount);`](https://github.com/jbx-protocol/juice-contracts-v2-code4rena/blob/733810a0339a5c0cb608345e6fc66a6edeac13cc/contracts/JBController.sol#L743)
- [L785: `_processedTokenTrackerOf[_projectId] = int256(tokenStore.totalSupplyOf(_projectId));`](https://github.com/jbx-protocol/juice-contracts-v2-code4rena/blob/733810a0339a5c0cb608345e6fc66a6edeac13cc/contracts/JBController.sol#L785)
- [L859: `_processedTokenTrackerOf[_projectId] = int256(_totalTokens + tokenCount);`](https://github.com/jbx-protocol/juice-contracts-v2-code4rena/blob/733810a0339a5c0cb608345e6fc66a6edeac13cc/contracts/JBController.sol#L859)

> Note: in the [L1076](https://github.com/jbx-protocol/juice-contracts-v2-code4rena/blob/733810a0339a5c0cb608345e6fc66a6edeac13cc/contracts/JBController.sol#L1076) and [L1077](https://github.com/jbx-protocol/juice-contracts-v2-code4rena/blob/733810a0339a5c0cb608345e6fc66a6edeac13cc/contracts/JBController.sol#L1077) there are two more casts but in the [L1075](https://github.com/jbx-protocol/juice-contracts-v2-code4rena/blob/733810a0339a5c0cb608345e6fc66a6edeac13cc/contracts/JBController.sol#L1075) check the cast

### Tools Used

Review

### Recommended Mitigation Steps

Use a SafeCast library of openzeppelin [`toUint256(int256 value)` and `toInt256(uint256 value)`](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/8c49ad74eae76ee389d038780d407cf90b4ae1de/contracts/utils/math/SafeCast.sol) or check the number before cast it


