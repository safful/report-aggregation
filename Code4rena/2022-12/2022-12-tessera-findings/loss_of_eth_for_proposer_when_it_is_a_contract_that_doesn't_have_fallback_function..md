## Tags

- bug
- 2 (Med Risk)
- disagree with severity
- primary issue
- satisfactory
- selected for report
- sponsor confirmed
- M-07

# [Loss of ETH for proposer when it is a contract that doesn't have fallback function.](https://github.com/code-423n4/2022-12-tessera-findings/issues/40) 

# Lines of code

https://github.com/code-423n4/2022-12-tessera/blob/f37a11407da2af844bbfe868e1422e3665a5f8e4/src/seaport/modules/OptimisticListingSeaport.sol#L209


# Vulnerability details

## Description

sendEthOrWeth() is used in several locations in OptimisticListingSeaport:
1. rejectProposal - sent to proposer
2. rejectActive - sent to proposer
3. cash - sent to msg.sender

This is the implementation of sendEthOrWeth:
```
function _attemptETHTransfer(address _to, uint256 _value) internal returns (bool success) {
    // Here increase the gas limit a reasonable amount above the default, and try
    // to send ETH to the recipient.
    // NOTE: This might allow the recipient to attempt a limited reentrancy attack.
    (success, ) = _to.call{value: _value, gas: 30000}("");
}
/// @notice Sends eth or weth to an address
/// @param _to Address to send to
/// @param _value Amount to send
function _sendEthOrWeth(address _to, uint256 _value) internal {
    if (!_attemptETHTransfer(_to, _value)) {
        WETH(WETH_ADDRESS).deposit{value: _value}();
        WETH(WETH_ADDRESS).transfer(_to, _value);
    }
}
```

The issue is that the receive could be a contract that does not have a fallback function. In this scenario, \_attemptETHTransfer will fail and WETH would be transferred to the contract. It is likely that it bricks those funds for the contract as there is no reason it would support interaction with WETH tokens. 

It can be reasonably assumed that developers will develop contracts which will interact with OptimisticListingSeaport using proposals. They are not warned and are likely to suffer losses.

## Impact

Loss of ETH for proposer when it is a contract that doesn't have fallback function.

## Tools Used

Manual audit

## Recommended Mitigation Steps

Either enforce that proposer is an EOA or take in a recipient address for ETH transfers.