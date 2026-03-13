---
severity: [H-3]
status: Pending
affected-contracts: TSwapPool.sol
---
**Title:**

No slippage protection in the `swapExactOutput` function, making it vulnerable to sandwich attacks

**Description:**

Slippage protection is important to protect users from trading at unpredictable prices. This is usually a medium issue, but in our case, combined with the [H-2] issue it is a critical vulnerability. The price formula is wrong when calculating the input amount with `getInputAmountBasedOnOutput`. The input amount will be way higher than it should be. Without slippage protection, the transaction will pass if the user has enough tokens and approval.
This also exposes users to sandwich attacks.

**Impact:**

Users will pay way more than expected for a swap.

**Proof Of Concept:**

*Actors:*
- Attacker: A liquidity provider performing a sandwich attack on User
- User: Normal user

*Description:*
1) The attacker owns the whole liquidity pool assets
2) The Attacker waits for a user to swap with the `swapExactOutput` function.
3) The Attacker buys pool tokens just before the User
4) The User buys pool tokens and pays a lot of WETH
5) The Attacker sells its pool tokens with sandwich attack profit + wrong fee formula profits

*Proof of Code:*

```javascript
    function test_sandwichAttack() public {
        // ADD LIQUIDITY

        vm.startPrank(liquidityProvider);

        poolToken.mint(liquidityProvider, 100e18);
        weth.mint(liquidityProvider, 10e18);

        poolToken.approve(address(pool), 100e18);
        weth.approve(address(pool), 10e18);

        pool.deposit(
            10e18,
            1e18,
            100e18,
            uint64(block.timestamp + 30)
        );

        // SANDWICH BUY

        uint256 inputAmount = pool.getInputAmountBasedOnOutput(
            1e17,
            weth.balanceOf(address(pool)),
            poolToken.balanceOf(address(pool))
        );
        weth.mint(liquidityProvider, inputAmount);
        weth.approve(address(pool), inputAmount);

        uint256 expectedOutputAmount = pool.getOutputAmountBasedOnInput(
            inputAmount,
            weth.balanceOf(address(pool)),
            poolToken.balanceOf(address(pool))
        );
        console2.log(expectedOutputAmount);

        pool.swapExactOutput(
            weth,
            poolToken,
            1e17,
            uint64(block.timestamp + 30)
        );

        vm.stopPrank();

        // USER IS BUYING

        vm.startPrank(user);

        inputAmount = pool.getInputAmountBasedOnOutput(
            1e16,
            weth.balanceOf(address(pool)),
            poolToken.balanceOf(address(pool))
        );
        weth.mint(user, inputAmount);
        weth.approve(address(pool), inputAmount);

        uint256 actualOutputAmount = pool.getOutputAmountBasedOnInput(
            inputAmount,
            weth.balanceOf(address(pool)),
            poolToken.balanceOf(address(pool))
        );
        console2.log(actualOutputAmount);

        pool.swapExactOutput(
            weth,
            poolToken,
            1e16,
            uint64(block.timestamp + 30)
        );

        vm.stopPrank();

        // SANDWICH SELL

        vm.startPrank(liquidityProvider);

        inputAmount = pool.getInputAmountBasedOnOutput(
            1e17,
            weth.balanceOf(address(pool)),
            poolToken.balanceOf(address(pool))
        );
        weth.mint(liquidityProvider, inputAmount);
        weth.approve(address(pool), inputAmount);

        pool.swapExactOutput(
            weth,
            poolToken,
            1e17,
            uint64(block.timestamp + 30)
        );

        vm.stopPrank();

        assertGt(expectedOutputAmount, actualOutputAmount);
    }
```

**Recommended Mitigation:**

Add a `maxInputAmount` parameter to the `swapExactOutput` function:

```diff
+    error TSwapPool__InputTooHigh(uint256 actual, uint256 max);

    ...

    function swapExactOutput(
        IERC20 inputToken,
+        uint256 maxInputAmount
        IERC20 outputToken,
        uint256 outputAmount,
        uint64 deadline
    )
        public
        revertIfZero(outputAmount)
        revertIfDeadlinePassed(deadline)
        returns (uint256 inputAmount)
    {
        uint256 inputReserves = inputToken.balanceOf(address(this));
        uint256 outputReserves = outputToken.balanceOf(address(this));

        inputAmount = getInputAmountBasedOnOutput(
            outputAmount,
            inputReserves,
            outputReserves
        );

+        if (inputAmount > maxInputAmount) {
+            revert TSwapPool__InputTooHigh(inputAmount, maxInputAmount);
+        }

        _swap(inputToken, inputAmount, outputToken, outputAmount);
    }
```