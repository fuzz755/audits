---
severity: [M-2]
status: Pending
affected-contracts: TSwapPool.sol
---
**Title:**

The `_addLiquidityMintAndTransfer` function doesn't follow CEI pattern

**Description:**

When the `_addLiquidityMintAndTransfer` function is called, the user is receiving newly minted liquidity tokens before sending the assets to the pool. This is not complying with CEI pattern which can lead to unexpected behaviors with ERC-777 tokens that allow reentrency.

**Impact:**

Reentrency risk.

**Recommended Mitigation:**

Follow CEI pattern

```diff
    ) private {
-       _mint(msg.sender, liquidityTokensToMint);
-       emit LiquidityAdded(msg.sender, poolTokensToDeposit, wethToDeposit);

        // Interactions
        i_wethToken.safeTransferFrom(msg.sender, address(this), wethToDeposit);
        i_poolToken.safeTransferFrom(
            msg.sender,
            address(this),
            poolTokensToDeposit
        );

+       _mint(msg.sender, liquidityTokensToMint);
+       emit LiquidityAdded(msg.sender, poolTokensToDeposit, wethToDeposit);
    }
```