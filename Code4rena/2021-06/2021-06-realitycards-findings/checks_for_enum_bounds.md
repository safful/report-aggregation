## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed
- resolved

# [Checks for enum bounds](https://github.com/code-423n4/2021-06-realitycards-findings/issues/9) 

# Handle

gpersoon


# Vulnerability details

## Impact
For the enums Mode and State, checks are made that the variables are within bounds. Here specific size are used, e.g. 2 and 4.
If the size of the enums would be changed in the future, those numbers don't change automatically.
Also solidity provides in-built check to check that variables are within bounds, which could be used instead. This also make the code more readable.

## Proof of Concept
// https://github.com/code-423n4/2021-06-realitycards/blob/main/contracts/RCMarket.sol#L202
enum Mode {CLASSIC, WINNER_TAKES_ALL, SAFE_MODE}
function initialize(
     ...
        assert(_mode <= 2);        // can be removed
      ...
         mode = Mode(_mode);  // this makes sure: 0<=mode<=2   // move to top

https://github.com/code-423n4/2021-06-realitycards/blob/main/contracts/interfaces/IRCMarket.sol#L7
    enum States {CLOSED, OPEN, LOCKED, WITHDRAW}

// https://github.com/code-423n4/2021-06-realitycards/blob/main/contracts/RCMarket.sol#L1094
    function _incrementState() internal {
        assert(uint256(state) < 4);                 // can be removed
        state = States(uint256(state) + (1));  // this makes sure: 0<=state<=3
        emit LogStateChange(uint256(state));
    }

## Tools Used

## Recommended Mitigation Steps
For function initialize:
Remove the "assert(_mode <= 2);" and move the statement "mode = Mode(_mode);" to the top of the function and add a comment

For function _incrementState: 
Remove "assert(uint256(state) < 4);" and add a comment at "state = States(uint256(state) + (1));"

