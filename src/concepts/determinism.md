# Determinism

From [Wikipedia](https://en.wikipedia.org/wiki/Deterministic_algorithm): A deterministic algorithm is an algorithm that, given a particular input, will always produce the same output.

Applying this definition to Borger:

- The `simulation_loop` that you write, as a whole, is assumed to be a deterministic algorithm by the underlying engine that calls it. This is an invariant you must uphold.
- The "input" of the algorithm (not be confused with [input state](./io-state.md)) is the initial state of the `GameContext` object when `simulation_loop` first begins.
- The "output" of the algorithm (again, not to be confused with [output state](./io-state.md)) is the mutated state of the `GameContext` object when `simulation_loop` finishes running.

```
1. INPUT:     GameContext starts out in some kinda way
      ↓
2. ALGORITHM: simulation_loop does stuff to GameContext
      ↓
3. OUTPUT:    GameContext is different now
```

_The same input should always produce the same output._

While this determinism requirement may sound scary and complicated at first, in practice, it simply means avoiding certain patterns that produce unpredictable results. These are the main 3 offenders you're most likely to run into:

1. Borger's API is the only reliable source of truth for understanding when a tick has occurred, because the system clock continues to move forward even when Borger [**rolls back**](./rollback.md).
   - ❌ `std::time`
   - ❌ `web_time` crate
   - ❌ `chrono` crate
   - ✅ `TickInfo::id()`
2. If using the `rand` crate (or similar) for random number generation, always use a [seed](https://en.wikipedia.org/wiki/Random_seed) derived from game state:
   - ❌

     ```rust
     let mut rng = thread_rng();
     let n: f32 = rng.gen_range(0.0..10.0);

     //ironically thread_rng() is TOO random, and returns something
     //different every time, even if the tick ID hasn't changed
     ```

   - ✅

     ```rust
     let mut rng = SmallRng::seed_from_u64(ctx.tick.id() ^ 2468);
     let n: f32 = rng.gen_range(0.0..10.0);

     //2468 can be replaced with any random constant, or even a game
     //state variable, in order to avoid getting the same result from
     //every SmallRNG instance in the same tick
     ```

3. If using Rust's built-in `HashMap` or `HashSet`, keep in mind that the iteration order is randomized, so deterministic code can't rely on them to iterate in any certain order. You can still use them and their iterators, but whatever value you're attempting to derive from them must still be deterministic. For example:
   - ✅ You could safely calculate the sum of a `HashSet<i32>` by iterating it, because _integer_ addition is commutative.
   - ❌ On the other hand, `hash_set.iter().next()` is bad because it essentially returns a random value.
   - ✅ `BTreeMap` and `BTreeSet` DO have deterministic iteration order, but have different performance characteristics.

Historically, floating-point arithmetic (working with fractional, non-integer numbers) has caused a lot of non-deterministic grief for multiplayer games. For example, `f32::consts::PI.sin()` might return a slightly different number depending on whether it runs on an ARM or Intel CPU, meaning players on the same server who have different hardware don't see exactly the same thing. With Borger, this is no longer an issue, because the same WASM binary runs on all CPU's. [The official spec guarantees cross-platform determinism](https://webassembly.github.io/spec/core/exec/numerics.html#floating-point-operations) for all situations that a game would care about.

Also note that determinism comes in varying degrees of strictness, depending on the [**trade-off**](./trade-offs.md) used.
