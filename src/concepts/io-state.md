# Input and Output State

_State_ is the data that is available to the game to read, process, and write to.

- In both Rust and TypeScript, it takes the form of a big chungus struct whose shape is defined entirely by your [`state.ts`](../api/state.md).
- Networking and synchronization happen automagically whenever you mutate state. A game written with Borger never ever manually sends data over the wire.
- State is susceptible to [**mispredictions**](./rollback-and-misprediction.md#misprediction).

### Input

_Input state_ holds information such as mouse cursor position, analog stick angle, whether certain controller buttons are currently pressed down, and more. It is the sole means for a [**client**](./server-and-client.md#client) to interact with the game session.

- It is only writable during a [**presentation tick**](./simulation-and-presentation.md#presentation).
- Each client owns their own input state object that they are individually responsible for populating.
- At the start of each presentation tick, The `Borger.Input` object starts out in the default, empty state, and must be populated by the end of the tick.
- Your presentation loop fills it out like a form using standard DOM input events: `mousemove`, `touchstart`, `keydown`, etc.

### Output

_Output state_ describes everything in the virtual game world: player positions, health, door hinge angles, mission objectives, and more.

- It is only writable during a [**simulation tick**](./simulation-and-presentation.md#simulation).
- In simulation logic, the output struct is called `SimulationState`. Clients' inputs are nested inside each individual client struct for convenience. In presentation, it's simply known as `Borger.Output`.
- The [**server and client**](./server-and-client.md) both share a copy of output state. The server has a complete view of everything, while the client can only see what's in [**scope**](./clients-and-scopes.md).
