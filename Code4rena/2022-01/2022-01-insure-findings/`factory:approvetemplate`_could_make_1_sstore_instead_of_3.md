## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [`Factory:approveTemplate` could make 1 SSTORE instead of 3](https://github.com/code-423n4/2022-01-insure-findings/issues/298) 

# Handle

Dravee


# Vulnerability details

## Impact
Increased gas cost as SSTOREs are very expensive

## Proof of Concept
The code is as follows :
```
094:     function approveTemplate(
095:         IUniversalMarket _template,
096:         bool _approval,
097:         bool _isOpen,
098:         bool _duplicate
099:     ) external override onlyOwner {
100:         require(address(_template) != address(0));
101:         templates[address(_template)].approval = _approval; //@audit-info SSTORE
102:         templates[address(_template)].isOpen = _isOpen; //@audit-info SSTORE
103:         templates[address(_template)].allowDuplicate = _duplicate; //@audit-info SSTORE
104:         emit TemplateApproval(_template, _approval, _isOpen, _duplicate);
105:     }
```
As we can see, it's making 3 SSTORE operations, one for each boolean. The code could be optimized as follows to save gas :
```
    function approveTemplate(
        IUniversalMarket _template,
        bool _approval,
        bool _isOpen,
        bool _duplicate
    ) external override onlyOwner {
        require(address(_template) != address(0));
        Template memory approvedTemplate = new Template(_isOpen, _approval, _duplicate);
        templates[address(_template)] = approvedTemplate; //@audit-info only one SSTORE
        emit TemplateApproval(_template, _approval, _isOpen, _duplicate);
    }
```

## Tools Used
VS Code

## Recommended Mitigation Steps
Use a memory `Template ` struct and write in storage only once

