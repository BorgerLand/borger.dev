# Trade-Offs

The _trade-offs_ annotation system is the heart of what makes Borger unique. All [**simulation**](./simulation-and-presentation.md#simulation) logic grapples with the tension between [**latency**](../#the-problem), [**state visibility**](./clients-and-scopes.md), and [**misprediction**](./rollback-and-misprediction.md#misprediction) risk. Trade-offs allow you to explicitly choose how this tension affects your code (and thus the overall game feel), on a spectrum between "responsive+potentially incorrect" to "laggy+definitely correct". They are applied as wrappers around functions and other blocks of code, so a quick glance at the start of a block tells you everything you need to know about the multiplayer characteristics of any game mechanic.

### Comparison Table

<style>
  table {
    border-collapse: separate;
    border-spacing: 0;
  }

  table td:not(:nth-child(1)), table th:not(:nth-child(1)) {
    min-width: 400px;
    border-left: none !important;
  }

  table td:nth-child(1), table th:nth-child(1) {
    position: sticky;
    left: 0;
  }

  table tr:not(:nth-child(1)) td:nth-child(1) {
    border-right: 1px solid var(--table-border-color);
  }

  table tr:nth-child(odd) td:nth-child(1) {
    background: var(--bg);
  }

  table tr:nth-child(even) td:nth-child(1) {
    background: var(--table-alternate-bg);
  }
</style>

|                           | **Rule of Thumb**                                 | **Where**                                                                                                                                                                                                                                                                        | **Latency/Responsiveness**                                                                                                                                                                                                                                    | **Correctness**                                                                                                                                                                       | **State Visibility**                                                                                                                                             | **[Determinism](./determinism.md) Requirements**                                                                              | **Example Use Cases**                                                                                                                                                                                                                                                                                                     |
| ------------------------- | ------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **`Immediate` (default)** | Immediate response without waiting for the server | Code runs on both [**server**](./server-and-client.md#server) and [**client**](./server-and-client.md#client). Use of Immediate mode does NOT give clients any [**cheating**](./cheating.md) ability or authority over what everyone else sees - it's simply a local prediction. | Instantaneous. Press a button on the client, see result on screen immediately without waiting for server reply.                                                                                                                                               | Highest risk of [**mispredction**](./misprediction.md) due to not having access to all relevant [**state**](./io-state.md). The server may produce different results than the client. | Can only access [**client-visible state**](./clients-and-scopes.md). Accessing private state causes a compile error; out-of-scope access causes a runtime error. | Must be deterministic between the server and any clients executing it. They should independently arrive at the same result.   | Processing client inputs (movement, shooting, etc.) should happen here in Immediate as much as possible for best game feel. Controls are by far the most latency-sensitive aspect of gameplay. Any physics interactions should also be immediate whenever possible: damage-on-collision mechanics, moving platforms, etc. |
| **`WaitForServer`**       | Hide sensitive, private data from clients         | Code runs only on the [**server**](./server-and-client.md#server)                                                                                                                                                                                                                | Slower if used for processing client inputs/interactions (their individual RTT), otherwise feels smooth. The server executes this regardless of whether it has received inputs from all clients, and clients will render the result whenever they receive it. | If it is possible for a client's input to change the results (eg. someone stepping in front of an NPC), there is a risk of [**mispredction**](./misprediction.md).                    | Has full access to all game state, including private/hidden data.                                                                                                | Must only be locally deterministic (consistent results on the same device across multiple runs, but may vary between devices) | Logic that affects entities seen by clients but are unable to be predicted by clients due to having some private state (NPC)                                                                                                                                                                                              |
| **`WaitForConsensus`**    | Guaranteed correctness at all costs               | Code runs only on the [**server**](./server-and-client.md#server)                                                                                                                                                                                                                | Slowest. The server waits to receive all inputs from all clients (up to 1.5 seconds before timing out) before executing. Clients won't see the result until RTT of the slowest client.                                                                        | Always correct. No [**mispredction**](./misprediction.md).                                                                                                                            | Has full access to all game state, including private/hidden data.                                                                                                | Determinism is not required because this code will only ever run once.                                                        | Backend calls (updating the leaderboard in the database), large/important game state change events that would look horrible if rolled back/mispredicted (game over, level change)                                                                                                                                         |

### Identification

Trade-offs are implemented as a no-op generic parameter on the `DiffSerializer` struct (and `GameContext` by extension), which forces any function that mutates [**output state**](./io-state.md#output) to specify under which trade-off context it can execute. This is what's meant by "annotations": you see trade-offs at the top of every function signature and every `multiplayer_tradeoff!()` call, lending an understanding to the multiplayer implications when it runs.

In this example, note the `WaitForConsensus` in the signature. It immediately (pun intended) tells you that this function is non-deterministic, never rolls back, and only happens on the server (from the information in the above table)

```rust
pub fn on_client_connect
(
	state: &mut SimulationState,
	client_id: usize32,
	tick_id: TickID,
	diff: &mut DiffSerializer<WaitForConsensus>,
)
```

And the simulation loop of course is tagged with the most flexible trade-off, `Immediate`.

```rust
fn simulation_loop(ctx: &mut GameContext<Immediate>)
```

Sometimes it can make sense for a function to be called under multiple contexts:

```rust
//a character may be client-controlled (Immediate) or an NPC (WaitForServer)
fn update_character(ctx: &mut GameContext<impl ImmediateOrWaitForServer>)
```

### [`multiplayer_tradeoff!()`](../api/rust.md#/borger/macro.multiplayer_tradeoff.html)

This macro is what allows toggling between trade-offs. Notably, **trade-offs can only be nested in order of increasing latency**:

```rust
fn simulation_loop(ctx: &mut GameContext<Immediate>)
{
	multiplayer_tradeoff!(WaitForServer,
	{
		multiplayer_tradeoff!(WaitForConsensus,
		{
			//game logic here

			//(you can skip directly from Immediate to WaitForConsensus
			//if needed. this is demonstration only)
		});
	});
}
```

### `#[server]`

Functions who are meant to only be called in `WaitForServer` and `WaitForConsensus` contexts must be tagged with this attribute macro, in order to strip the code from the client build. Ideally the compiler could flag this automatically through static analysis, but for now it must be manually inserted in this early release of Borger. Forgetting to include it could result in a failed client build, a client-sided runtime crash if `unwrap()`ing on [**out-of-scope**](./clients-and-scopes.md) state, or it may just work normally if the function doesn't attempt to read private state.
