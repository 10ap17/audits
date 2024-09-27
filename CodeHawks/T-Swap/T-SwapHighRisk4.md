# Mismatched Input and Output Tokens in`sellPoolTokens`            

### Relevant GitHub Links

https://github.com/Cyfrin/2024-06-t-swap/blob/main/src/TSwapPool.sol

## Summary
The `sellPoolTokens` function in the `TSwapPool` contract mismatches the input and output tokens, resulting in `users` receiving incorrect amounts of tokens. This error occurs because the function uses `swapExactOutput` instead of the correct `swapExactInput` function. `Users` specify the exact amount of input tokens they want to sell, not the output tokens they want to receive.

## Vulnerability Details
The` sellPoolToken`s function is intended to allow `users` to sell their pool tokens and receive `WETH` in return. `Users` specify the number of pool tokens they wish to sell using the `poolTokenAmount` parameter. However, the function miscalculates the swapped amount due to using the `swapExactOutput` function instead of the `swapExactInput` function. As a result, `users` swap an incorrect amount of tokens, disrupting the intended functionality of the protocol.
Current implementation:

```solidity
function sellPoolTokens(uint256 poolTokenAmount) external returns (uint256 wethAmount) {
    return swapExactOutput(i_poolToken, i_wethToken, poolTokenAmount, uint64(block.timestamp));
}
```
Corrected implementation:

```solidity
function sellPoolTokens(
        uint256 poolTokenAmount
    ) external returns (uint256 wethAmount) {
        return
            swapExactOutput(
                i_poolToken,
                i_wethToken,
                poolTokenAmount,
                uint64(block.timestamp)
            );
    }
```
## Impact
1. `Incorrect Token Swaps`: `Users` receive incorrect amounts of `WETH` when selling pool tokens, causing financial discrepancies.
2. `Severe Disruption`: The functionality of the protocol is severely disrupted, leading to potential loss of `use`r trust and confidence.
3. `User Confusion`: `Users` may become confused and frustrated due to the unexpected behavior of the `sellPoolTokens` function.

## Tools Used
1. Manual Code Review

## Recommendations
To fix this vulnerability, the implementation of the `sellPoolTokens` function should be changed to use the `swapExactInput` function instead of the `swapExactOutput` function. Additionally, a new parameter `minWethToReceive` should be added to ensure users receive at least the minimum expected amount of `WETH`.

Corrected code:

```diff
function sellPoolTokens(uint256 poolTokenAmount, uint256 minWethToReceive) external returns (uint256 wethAmount) {
-     return swapExactOutput( i_poolToken,i_wethToken, poolTokenAmount, uint64(block.timestamp));
+    return swapExactInput(i_poolToken, poolTokenAmount, i_wethToken, minWethToReceive, uint64(block.timestamp));
}
```