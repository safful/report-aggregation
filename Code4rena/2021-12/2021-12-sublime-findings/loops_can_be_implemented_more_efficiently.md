## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Loops can be implemented more efficiently](https://github.com/code-423n4/2021-12-sublime-findings/issues/157) 

# Handle

0x0x0x


# Vulnerability details

## Proof of Concept

Example:

```

for (uint i = 0; i < arr.length; i++) {

//Operations not effecting the length of the array.

}

```

Loading length of array costs gas. Therefore, the length should be cached, if the length of the array doesn't change inside the loop. Furthermore, there is no need to assign the initial value 0. This costs extra gas.

Recommended implementation:

```

uint length = arr.length;

for (uint i; i < length; ++i) {

//Operations not effecting the length of the array.

}

```

By doing so the length is only loaded once rather than loading it as many times as iterations (Therefore, less gas is spent).

## Occurences

```

./CreditLine/CreditLine.sol:484:        for (uint256 _index = 0; _index < _strategyList.length; _index++) {
./CreditLine/CreditLine.sol:662:        for (uint256 _index = 0; _index < _strategyList.length; _index++) {
./CreditLine/CreditLine.sol:738:        for (uint256 _index = 0; _index < _strategyList.length; _index++) {
./CreditLine/CreditLine.sol:892:        for (uint256 index = 0; index < _strategyList.length; index++) {
./CreditLine/CreditLine.sol:959:        for (uint256 index = 0; index < _strategyList.length; index++) {
./SavingsAccount/SavingsAccount.sol:289:        for (uint256 i = 0; i < _strategyList.length; i++) {
./SavingsAccount/SavingsAccount.sol:467:        for (uint256 i = 0; i < _strategyList.length; i++) {

```

