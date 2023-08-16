## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [prevent burn in _transfer](https://github.com/code-423n4/2021-07-sherlock-findings/issues/29) 

# Handle

gpersoon


# Vulnerability details

## Impact
The function _transfer in SherXERC20.sol allow transfer to address 0.
This is usually considered the same as burning the tokens and the Emit is indistinguishable from an Emit of a burn.

However the burn function in LibSherXERC20.sol has extra functionality, which _transfer doesn't have.
sx20.totalSupply = sx20.totalSupply.sub(_amount);

So it is safer to prevent _transfer to address 0 (which is also done in the openzeppelin erc20 contract)
See: https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC20/ERC20.sol#L226

Note: minting from address 0 will not work because that is blocked by the safemath sub in:
 sx20.balances[_from] = sx20.balances[_from].sub(_amount);

## Proof of Concept
https://github.com/code-423n4/2021-07-sherlock/blob/main/contracts/facets/SherXERC20.sol#L118
function _transfer(address _from, address _to, uint256 _amount) internal {
    SherXERC20Storage.Base storage sx20 = SherXERC20Storage.sx20();
    sx20.balances[_from] = sx20.balances[_from].sub(_amount);
    sx20.balances[_to] = sx20.balances[_to].add(_amount);
    emit Transfer(_from, _to, _amount);
  }

// https://github.com/code-423n4/2021-07-sherlock/blob/main/contracts/libraries/LibSherXERC20.sol#L29
function burn(address _from, uint256 _amount) internal {
    SherXERC20Storage.Base storage sx20 = SherXERC20Storage.sx20();
    sx20.balances[_from] = sx20.balances[_from].sub(_amount);
    sx20.totalSupply = sx20.totalSupply.sub(_amount);
    emit Transfer(_from, address(0), _amount);
  }
## Tools Used

## Recommended Mitigation Steps
add something like to following to _transfer of SherXERC20.sol:
        require(_to!= address(0), "Transfer to the zero address");

Or update sx20.totalSupply if burning a desired operation.

