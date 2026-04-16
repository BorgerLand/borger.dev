# Input and Output State

_State_ is the data that is available to the game to read, process, and write to.

- In both Rust and TypeScript, it takes the form of a big chungus struct whose shape is defined by your [`state.ts`](../api/state.md).
- Networking and synchronization happen automagically whenever you mutate state. A game written with Borger never ever manually sends data over the wire.

### Input

_Input state_ is the sole means for a [**client**](./server-and-client.md#client) to interact with the game session.

- It is only writable during a [**presentation tick**](./simulation-and-presentation.md#presentation).
- Each client owns their own input state object that they are individually responsible for populating.
- At the start of each presentation tick, The `Borger.Input` object starts out in the default, empty state, and must be populated by the end of the tick.
- Your presentation loop fills it out like a form using standard DOM input events: `mousemove`, `touchstart`, `keydown`, etc.

### Output

_Output state_ describes everything in the game world: player positions, health, door hinge angles, mission objectives, and more. Any information that needs to be kept in sync belongs here.

- It is only writable during a [**simulation tick**](./simulation-and-presentation.md#simulation).
-
