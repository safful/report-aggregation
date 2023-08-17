## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed
- selected for report
- responded

# [ApprovalAll event is missing parameters](https://github.com/code-423n4/2022-10-holograph-findings/issues/270) 

# Lines of code

https://github.com/code-423n4/2022-10-holograph/blob/f8c2eae866280a1acfdc8a8352401ed031be1373/src/enforcer/HolographERC721.sol#L392


# Vulnerability details

## Impact
beforeApprovalAll() / afterApprovalAll() can only pass "to" and "approved", missing "owner", if contract listening to this event,but does not know who approve it, so can not react to this event
Basically, this event cannot be used

## Proof of Concept
```
  function setApprovalForAll(address to, bool approved) external {
....

    if (_isEventRegistered(HolographERC721Event.beforeApprovalAll)) {
      require(SourceERC721().beforeApprovalAll(to, approved)); /***** only to/approved ,need owner
    }  

    _operatorApprovals[msg.sender][to] = approved;

    if (_isEventRegistered(HolographERC721Event.afterApprovalAll)) {
      require(SourceERC721().afterApprovalAll(to, approved)); /***** only to/approved ,need owner
    }
  }
```

## Tools Used

## Recommended Mitigation Steps


add parameter: owner 

```
interface HolographedERC721 {
...

- function beforeApprovalAll(address _to, bool _approved) external returns (bool success);
+ function beforeApprovalAll(address owner, address _to, bool _approved) external returns (bool success);

- function afterApprovalAll(address _to, bool _approved) external returns (bool success);
+ function afterApprovalAll(address owner, address _to, bool _approved) external returns (bool success);
```

```
  function setApprovalForAll(address to, bool approved) external {

    if (_isEventRegistered(HolographERC721Event.beforeApprovalAll)) {
-     require(SourceERC721().beforeApprovalAll(to, approved)); 
+     require(SourceERC721().beforeApprovalAll(msg.sender,to, approved)); 
    }  

    _operatorApprovals[msg.sender][to] = approved;

    if (_isEventRegistered(HolographERC721Event.afterApprovalAll)) {
-      require(SourceERC721().afterApprovalAll(to, approved));
+      require(SourceERC721().afterApprovalAll(msg.sender,to, approved));
    }
  }
```