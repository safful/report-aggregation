## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [receive function](https://github.com/code-423n4/2021-10-slingshot-findings/issues/94) 

# Handle

pauliax


# Vulnerability details

## Impact
Slingshot contract does not need a 'receive' function as it is not supposed to receive ETH directly. Executioner has this function too and it needs to receive ETH from the WETH contract. Because it expects only WETH to send the native asset directly, it should check that the msg.sender is actually WETH contract.

## Recommended Mitigation Steps
receive() external payable {
  require(msg.sender == wrappedNativeToken, "...");
}


