# Workflow

The [landing page](/) may be a bit goofy, but "outline, decree, engrave" is genuinely the recommended workflow. This page does not go into detail because each topic has its own section in the Concepts chapter. In slightly less arrogant terms, the overall intention for using Borger as a framework is:

1. **Outline**: Additions to the [**state**](../concepts/io-state.md) definition (defined in [`state.ts`](../api/state.md)) enable storage for new game mechanics that persist throughout a single game session.

2. **Decree**: This is [**simulation logic**](../concepts/simulation-and-presentation.md#simulation). Given the shape of data defined by `state.ts`, write the code that populates that state, governed by the rules of the game.

3. **Engrave**: [**Presentation logic**](../concepts/simulation-and-presentation.md#presentation) takes the populated state and decides how it should look and sound to the player.

Following these steps results in something that looks vaguely similar to a video game.
