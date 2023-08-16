## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [`permitAndMulticall()` May Be Used to Steal Funds Or as a Denial Of Service if `_from` Is Not The Message Sender](https://github.com/code-423n4/2022-02-pooltogether-findings/issues/20) 

# Lines of code

https://github.com/pooltogether/v4-twab-delegator/blob/2b6d42506187dd7096043e2dfec65fa06ab18577/contracts/PermitAndMulticall.sol#L46-L64
https://github.com/pooltogether/v4-twab-delegator/blob/2b6d42506187dd7096043e2dfec65fa06ab18577/contracts/PermitAndMulticall.sol#L31-L37
https://github.com/pooltogether/v4-twab-delegator/blob/2b6d42506187dd7096043e2dfec65fa06ab18577/contracts/TWABDelegator.sol#L438-L445


# Vulnerability details

## Impact

When the `_from` address is not the `msg.sender` `_multiCall()` will be made on behalf of the `msg.sender`. As a result each of the functions called by `multiCall()` will be made on behalf of `msg.sender` and not `_from`.

If functions such as `transfer()` or `unstake()` are called `msg.sender` will be the original caller which would transfer the attacker the funds if the `to` field is set to an attackers address.

Furthermore, if an attacker we to call `permitAndMulticall()` before the `_from` user they may use their signature and nonce combination. As a nonce is only allowe to be used once the siganture will no longer be valid and `_permitToken.permit()` will fail on the second call.

An attacker may use this as a Denial of Service (DoS) attack by continually front-running `permitAndCall()` using other users signatures.

## Proof of Concept

```
  function _multicall(bytes[] calldata _data) internal virtual returns (bytes[] memory results) {
    results = new bytes[](_data.length);
    for (uint256 i = 0; i < _data.length; i++) {
      results[i] = Address.functionDelegateCall(address(this), _data[i]);
    }
    return results;
  }
```

```
  function _permitAndMulticall(
    IERC20Permit _permitToken,
    address _from,
    uint256 _amount,
    Signature calldata _permitSignature,
    bytes[] calldata _data
  ) internal {
    _permitToken.permit(
      _from,
      address(this),
      _amount,
      _permitSignature.deadline,
      _permitSignature.v,
      _permitSignature.r,
      _permitSignature.s
    );

    _multicall(_data);
  }
```

## Recommended Mitigation Steps

Consider updating the `_from` field to be the `msg.sender` in `permitAndMulticall()` (or alternatively do this in `_permitAndMulticall()` to save some gas).

```
  function permitAndMulticall(
    uint256 _amount,
    Signature calldata _permitSignature,
    bytes[] calldata _data
  ) external {
    _permitAndMulticall(IERC20Permit(address(ticket)), msg.sender, _amount, _permitSignature, _data);
  }
```

