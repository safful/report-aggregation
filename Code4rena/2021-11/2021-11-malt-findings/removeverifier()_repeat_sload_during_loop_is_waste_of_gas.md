## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [removeVerifier() Repeat SLOAD During Loop Is Waste of Gas](https://github.com/code-423n4/2021-11-malt-findings/issues/117) 

# Handle

Meta0xNull


# Vulnerability details

## Impact
removeVerifier() loops follows this for-each pattern:
for (uint256 i = 0; i < array.length; i++) {
	// do something with `array[i]`
}

In such for loops, the array.length is read on every iteration, instead of caching it once in a local variable and read it again using the local variable.

## Proof of Concept
https://github.com/code-423n4/2021-11-malt/blob/main/src/contracts/TransferService.sol#L87-L88

## Tools Used
Manual Review

## Recommended Mitigation Steps
Read these values from memory once, cache them in local variables and then read them again using the local variables. For example:

Before:
for (uint i = 0; i < verifierList.length - 1; i = i + 1) {
if (verifierList[i] == _address) {

After:
uint256 verifierList_temp = verifierList

for (uint i = 0; i < verifierList_temp.length - 1; i = i + 1) {
if (verifierList_temp[i] == _address) {

