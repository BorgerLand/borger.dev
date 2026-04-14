# Input and Output State

_State_ is the data that is available to the game to read, process, and write to.

- It comes in the form of a big chungus `SimulationState` struct, whose shape is defined by your [`state.ts`](../api/state.md).
- Networking and synchronization happen automagically whenever you mutate state.

### Input

_Input state_ is the sole means for a [**client**](./server-and-client.md#client) to interact with the game session.

- It is only writable during a [**presentation tick**](./simulation-and-presentation.md#presentation).
- Each client owns their own input state object that they are individually responsible for populating.
- At the start of each presentation tick, The `Borger.Input` object starts out in the default, empty state, and must be populated by the end of the tick.

### Output

- It is only writable in simulation logic.
