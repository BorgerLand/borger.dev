# Rollback and Misprediction

### Misprediction

[**Latency**](../#the-problem) across the game session means that the [**simulation**](./simulation-and-presentation.md#simulation) must make _predictions_ in order for the game to feel responsive. And _predictions_, by definition, can be wrong.

Let's say the [**server**](./server-and-client.md#server) is chugging along as usual:

```
Tick ID 100
		 ↓
Tick ID 101
		 ↓
Tick ID 101
		 ↓
		etc.
```

And a [**client**](./server-and-client.md#client) is connected to that server, chugging alongside it in perfect sync:

```
Tick ID 100
		 ↓
Tick ID 101
		 ↓
Tick ID 101
		 ↓
		etc.
```

Now, let's say this particular client decides to press the spacebar key on tick ID 100 in order to jump. The game logic may look something like:

```rust
//(pseudocode)

if input.jump
{
	character.y += 10;
}
```

- When the client runs the simulation logic on tick ID 100, `input.jump == true`, because the client has direct access to their own keyboard. No slow internet connection is involved.
- When the server runs the simulation logic on tick ID 100, `input.jump == false`, because the spacebar key press is still making its way downtown over the internet. This is an example of _misprediction_.

The above misprediction is caused by the server having not received all [**inputs**](./io-state.md) from all clients yet. Clients also experience misprediction; arguably more so. Misprediction is fundamentally caused by the simulation loop's need to keep running and making predictions even when it doesn't have all the information it needs to be fully accurate. Its ability to continue blindly running forward is what allows inputs to feel responsive, which is very important in fast-paced multiplayer. The player would be very angry if jumping were to require [**waiting for consensus**](./trade-offs.md) across all devices. A platformer would be impossible to play.

In practice, what does misprediction look like to the player?

- Lag is the simplest and most common form of misprediction. Whether you're seeing other players stall and stutter during a bad connection, or a slight delay in picking up a weapon, the effects of lag are caused by your client being unable to predict something. It's waiting for the server to tell you what happened.
- Imagine 2 players trying to pick up the same gun at the same time. If the collision callback were to run in [**Immediate**](./trade-offs.md) mode, both of them equip the gun. This results in a gnarly misprediction when the server picks a winner and rips the gun back out of the loser's hand. Whether it's more important to quickly equip weapons or prevent mispredictions entirely depends on the type of game.

### Rollback

At its core, Borger is a _rollback_ framework. This is the idea that an older [**simulation tick ID**](./simulation-and-presentation.md#simulation) can rerun, when information pertaining to the past finally arrives over the slow internet connection.

- It occurs on both the [**server and client**](./server-and-client.md)
- It's a generic, brute-force, fix-all solution that looks not dissimilar to traveling back in time.
- It makes mispredictions much easier to reason about, at the cost of being difficult to implement. But you already installed Borger, so you'll do fine.

Building off the above jumping example in the misprediction section helps to convey how rollback steamrolls mispredictions out of existence:

1. Tick ID 100 executes on the server. `input.jump == false` and so `character.y` is untouched
2. Server keeps on simulating. Tick ID 100, 101, and 102 all come and go.
3. Server finally receives the jump input pertaining to tick ID 100.
4. 🚨😬🚨 Red alert. Server _rolls back_ to tick ID 100. Every single [**output state**](./io-state.md#output) change that happened in ticks 102, 101, and 100 are undone in reverse order.
5. Tick ID 100 re-executes on the server, this time with new a lease on life. `input.jump == true` now. Character jumps.
6. Server fasts forward back to where it should be. Tick ID 101 and 102 also re-execute.

So technically, each scheduled simulation tick that happens every 33 milliseconds actually consists of multiple ticks! In reality, the pristine timeline that the player is watching on their screen is nowhere near as linear and pretty as it seems.

They see this:

```
Tick ID 100
		 ↓   wait 33ms
Tick ID 101
		 ↓   wait 33ms
Tick ID 101
		 ↓   wait 33ms
		etc.
```

But what's actually happening underneath the hood is:

```
Tick ID 98
Tick ID 99
Tick ID 100
		 ↓   wait 33ms
Tick ID 99
Tick ID 100
Tick ID 101
		 ↓   wait 33ms
Tick ID 100
Tick ID 101
Tick ID 102
		 ↓   wait 33ms
		etc.
```

The engine makes sure that only the final sub-tick is ever presented to the player, and so as long as the code is [**deterministic**](./determinism.md), the illusion is complete and they are blissfully unaware of what's happening. It's important to note the performance implications here: more latency means more rollback means more ticks having to re-run during the short 33ms time window. The laggiest client hurts everyone.
