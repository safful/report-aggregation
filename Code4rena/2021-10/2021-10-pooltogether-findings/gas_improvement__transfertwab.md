## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed
- resolved

# [Gas improvement _transferTwab](https://github.com/code-423n4/2021-10-pooltogether-findings/issues/16) 

# Handle

gpersoon


# Vulnerability details

## Impact
I've found some gas improvements for _transferTwab, see below.

## Proof of Concept
https://github.com/pooltogether/v4-core/blob/master/contracts/Ticket.sol#L296-L313

## Tools Used

## Recommended Mitigation Steps

 function _transferTwab(address _from, address _to, uint256 _amount) internal {
        if (_from==_to) return; // no need to transfer if both are the same
        if (_from != address(0)) {
            _decreaseUserTwab(_from, _amount);
            if (_to == address(0)) _decreaseTotalSupplyTwab(_amount);
        }        
        if (_to != address(0)) {        
            _increaseUserTwab(_to, _amount);
            if (_from == address(0)) _increaseTotalSupplyTwab(_amount);
        }
    }

