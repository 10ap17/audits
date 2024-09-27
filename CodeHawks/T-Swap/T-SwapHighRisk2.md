# Invariant Violation            

### Relevant GitHub Links

https://github.com/Cyfrin/2024-06-t-swap/blob/main/src/TSwapPool.sol

## Summary
The `_swap` function in the `TSwapPool` contract breaks the protocol's invariant of `x * y = k `due to the extra tokens given to `users `after every `swapCount`. This invariant ensures that the product of the balances of the pool token (`x`) and `WETH` (`y`) remains constant (`k`). However, the additional incentive distributed in the `_swap` function disrupts this balance, leading to the potential draining of protocol funds over time.

## Vulnerability Details
The `TSwapPool` protocol adheres to a strict invariant of `x * y = k`, where x represents the balance of the pool token, y represents the balance of `WETH`, and `k` is a `constant`. This invariant is essential for maintaining the protocol's stability and ensuring fair exchanges. However, the following code block in the `_swap` function breaks this invariant:

```solidity
swap_count++;
if (swap_count >= SWAP_COUNT_MAX) {
    swap_count = 0;
    outputToken.safeTransfer(msg.sender, 1_000_000_000_000_000_000);
}
```
This block increments `swap_count` and, when `swap_count` reaches `SWAP_COUNT_MAX`, resets it to zero and transfers an extra `1 ETH` (`1,000,000,000,000,000,000 wei`) to the `user`. This additional transfer disrupts the balance between `x` and `y`, violating the `x * y = k` invariant and leading to an imbalance in the protocol.

## Impact
1. `Invariant Violation`: The protocol's fundamental invariant of `x * y = k` is broken, leading to potential imbalances in the pool.
2. `Funds Draining`: Over time, the extra tokens given to users can drain the `protocol's funds`, reducing liquidity and stability.
3. `Unfair Advantage`: `Users` can receive more tokens than they should, leading to an unfair advantage and potential exploitation.

## Tools Used
1. Manual Code Review

## Recommendations
To fix this vulnerability, remove the block of code that transfers extra tokens to users in the `_swap` function. Instead, consider implementing a different incentive mechanism that does not disrupt the `x * y = k` invariant. 