---
severity: [I-2]
status: Pending
affected-contracts: TSwapPool.sol
---
**Title:**

Use of magic numbers

**Description:**

The `getOutputAmountBasedOnInput` function is calculating a local variable using magic numbers to process fees: `997` and `1000`. These numbers should be constants and have a proper name describing what they do.

**Recommended Mitigation:**

Create constant variables in the contract storage:

```diff
    uint256 private constant MINIMUM_WETH_LIQUIDITY = 1_000_000_000;
    uint256 private swap_count = 0;
    uint256 private constant SWAP_COUNT_MAX = 10;
+    // divide FEE by FEE_PRECISION to get the true fee ratio
+    uint256 private constant FEE = 997;
+    uint256 private constant FEE_PRECISION = 1000;

...

-    uint256 inputAmountMinusFee = inputAmount * 997;
+    uint256 inputAmountMinusFee = inputAmount * FEE;
    uint256 numerator = inputAmountMinusFee * outputReserves;
-    uint256 denominator = (inputReserves * 1000) + inputAmountMinusFee;
+    uint256 denominator = (inputReserves * FEE_PRECISION) + inputAmountMinusFee;
    return numerator / denominator;
```