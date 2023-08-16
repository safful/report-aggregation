## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Unchecked transfers](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/31) 

# Handle

Reigada


# Vulnerability details

## Impact
Multiple calls to transferFrom and transfer are frequently done without checking the results. For certain ERC20 tokens, if insufficient tokens are present, no revert occurs but a result of “false” is returned. It’s important to check this. If you don’t, in this concrete case, some airdrop eligible participants could be left without their tokens. It is also a best practice to check this.

## Proof of Concept
AirdropDistributionMock.sol:132:        mainToken.transfer(msg.sender, claimable_to_send);
AirdropDistributionMock.sol:157:        mainToken.transfer(msg.sender, claimable_to_send);
AirdropDistribution.sol:542:        mainToken.transfer(msg.sender, claimable_to_send);
AirdropDistribution.sol:567:        mainToken.transfer(msg.sender, claimable_to_send);

InvestorDistribution.sol:132:        mainToken.transfer(msg.sender, claimable_to_send);
InvestorDistribution.sol:156:        mainToken.transfer(msg.sender, claimable_to_send);
InvestorDistribution.sol:207:        mainToken.transfer(msg.sender, bal);

Vesting.sol:95:        vestingToken.transferFrom(msg.sender, address(this), _amount);

PublicSale.sol:224:            mainToken.transfer(_member, v_value); 

## Tools Used
Manual testing

## Recommended Mitigation Steps
Check the result of transferFrom and transfer. Although if this is done, the contracts will not be compatible with non standard ERC20 tokens like USDT. For that reason, I would rather recommend making use of SafeERC20 library: safeTransfer and safeTransferFrom.

