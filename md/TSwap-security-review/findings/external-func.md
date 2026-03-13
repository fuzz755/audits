---
severity: [G-3]
status: Pending
affected-contracts: TSwapPool.sol
---
**Title:**

The `swapExactInput` function should be external

**Description:**

A public function that is never used inside of a smart contract can be external to consume less fees.

**Recommended Mitigation:**

Replace `public` by `external`:

```diff
    function swapExactInput(
        IERC20 inputToken,
        uint256 inputAmount,
        IERC20 outputToken,
        uint256 minOutputAmount,
        uint64 deadline
    )
-        public
+        external
        revertIfZero(inputAmount)
        revertIfDeadlinePassed(deadline)
```