---
severity: [G-1]
status: Pending
affected-contracts: PoolFactory.sol
---
**Title:**

The custom error `PoolFactory__PoolDoesNotExist` is declared but never used

**Description:**

This error is intended to be used inside of the `getToken` function, which is external. Using errors on external function is not useful. The error is not used anyway so it is just wasting gas.

**Recommended Mitigation:**

Delete the error:

```diff
-    error PoolFactory__PoolDoesNotExist(address tokenAddress);
```