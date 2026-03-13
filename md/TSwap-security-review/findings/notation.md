---
severity: [I-1]
status: Pending
affected-contracts: TSwapPool.sol
---
**Title:**

The variable `MINIMUM_WETH_LIQUIDITY` should use scientific notation

**Description:**

Scientific notation makes it easier to read big numbers, Ethereum is using 1e18 decimals and 1_000_000_000 can be written as 1e9.

**Recommended Mitigation:**

Use scientific notation:

```diff
-    uint256 private constant MINIMUM_WETH_LIQUIDITY = 1_000_000_000;
+    uint256 private constant MINIMUM_WETH_LIQUIDITY = 1e9;
```