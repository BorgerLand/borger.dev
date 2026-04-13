# Howdy. You've Reached Borger.

### The Problem

In a fast-paced multiplayer video game, **latency** is the root of all evil. The fundamental problem that you as a developer face is the highly variable amount of time between one player tapping a key and 10 other players seeing it.

Take, for example, a simple racing game in which 3 players drive their cars toward a self destruct button. The first player to cross the finish line explodes violently and wins the game as charred gears rain from the sky.

```
💥🏁             🚗💨
💥🏁             🚙💨
💥🏁             🏎️💨
```

However, if you were to sit the 3 players right next to each other and observe all 3 of their screens, you'd see something very odd:

_Player 1_

```
💥🏁        🚗💨
💥🏁             🚙💨
💥🏁             🏎️💨
```

_Player 2_

```
💥🏁             🚗💨
💥🏁        🚙💨
💥🏁             🏎️💨
```

_Player 3_

```
💥🏁             🚗💨
💥🏁             🚙💨
💥🏁        🏎️💨
```

You're witnessing the effects of _network latency_ (also known as _ping_ in gaming communities): the amount of time it takes for information to disseminate to all players across the game session.

- When player 1 presses the left arrow key, they see their car move immediately, and the game feels responsive to them.
- When player 2 presses the left arrow key, player 1 won't see any change for a fraction of a second.

Player 1 is looking at a delayed, outdated version of player 2's car. The delay is caused by how slow it is to send information, such as key presses, across the internet. Even players under the same roof have a small but noticeable amount of latency across their devices. Players on opposite sides of the globe are up against the speed of light and face a very high ping; perhaps 200-300 milliseconds and constantly fluctuating.

While the visual discrepancy is annoying enough on its own, it also wreaks havoc while writing game logic. What happens when all 3 players think they got to the finish line first? How do you make sure that only 1 player sees that they won? If 2 cars crash into each other, where do you render the dents? Your brain quickly melts and smells bad from trying to answer each individual question in the form of code.

### The Solution

**Borger** uses a technique called [rollback netcode](./concepts/rollback.md) to automatically resolve these types of conflicts. It allows writing multiplayer game logic with code that looks no different from single-player logic, solving one of the hardest problems in gamedev in a way that you don't even need to think about it.
