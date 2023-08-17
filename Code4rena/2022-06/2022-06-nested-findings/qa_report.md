## Tags

- bug
- QA (Quality Assurance)
- sponsor confirmed
- valid

# [QA Report](https://github.com/code-423n4/2022-06-nested-findings/issues/96) 

# [QA-1] Naming inconsistency - some arguments have ``_`` at their prefixes but others do not at NestedFactory.sol

Throughout the file ``NestedFactory.sol``, arguments of functions have ``_`` at their prefixes like ``function setFeeSplitter(FeeSplitter _feeSplitter)``. However, following 2 arguments do not have ``_`` at their prefixes which are not consistent.

https://github.com/code-423n4/2022-06-nested/blob/main/contracts/NestedFactory.sol#L121

https://github.com/code-423n4/2022-06-nested/blob/main/contracts/NestedFactory.sol#L133

---

# [QA-2] Use either ``_msgSender()`` or ``msg.sender`` 

Throughout the file ``NestedFactory.sol``, ``_msgSender()`` is used to get the sender. However, following 2 places use ``msg.sender`` which seem not consistent.

https://github.com/code-423n4/2022-06-nested/blob/main/contracts/NestedFactory.sol#L89

https://github.com/code-423n4/2022-06-nested/blob/main/contracts/NestedFactory.sol#L177

