## Tags

- bug
- disagree with severity
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [`updateOperator()` can be called before an operator is set in proxy](https://github.com/code-423n4/2022-05-aura-findings/issues/34) 

# Lines of code

https://github.com/code-423n4/2022-05-aura/blob/main/contracts/Aura.sol#L82


# Vulnerability details

## Impact
In `Aura.sol` the `updateOperator()` function can be called by anyone and it sets a new `operator` based on the address returned from `IStaker(vecrvProxy).operator()`.  The problem is that anyone can call this function even if the operator on `vecrvProxy` is not yet set.  If this is the case the operator in `Aura.sol` would be set to a zero address breaking the contract since functions like `init()` and `mint()` rely on the `msg.sender` being the `operator`. Even the `minterMint()` function relies on the `operator` since only the operator can set the `minter` which is the only one who can call `minterMinter()`. 

## Proof of Concept
https://github.com/code-423n4/2022-05-aura/blob/main/contracts/Aura.sol#L82

## Tools Used
Manual code review 

## Recommended Mitigation Steps
The `updateOperator()` function should not be a public function and should only be callable by an admin or the `operator` inside `Aura.sol`.  Also in the `updateOperator()` function,  there should be a check ensuring that the `newOperator` address is not a zero address to prevent breaking the contract by setting the `operator` to a zero address. 

