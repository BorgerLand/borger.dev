# Installation

### Hardware Requirements

Although games built with Borger are very small and efficient, running 2 `rustc` instances and `rust-analyzer` at the same time during development is absolutely brutal. At bare minimum, you will need at least 12GB of RAM and 10GB of available disk storage.

### Software Requirements

- [**Bash**](https://www.gnu.org/software/bash/): Required to run Borger's CLI tool. Installing it depends on your operating system:
  - Linux: Any self-respecting distribution should already have it. Borger itself was written on a 2014 Dell Precision M2800 equipped with [Xubuntu](https://xubuntu.org/download/).
  - macOS:
    1.  First you must [purchase](https://www.apple.com/mac/) expensive, unrepairable, unupgradable hardware
    2.  Check your Bash version in the terminal: `bash --version`. If it outputs version 3.2, congrats, your Bash is 20 years out of date. Otherwise, your Bash should be good to go already.
    3.  Before you can update it, you'll need [Homebrew](https://brew.sh/) installed.
    4.  `brew install bash`
    5.  Check the version again to be on the safe side. If it's still stuck at 3.2, restart the terminal.
  - Windows: Requires [Windows Subsystem for Linux (WSL)](https://learn.microsoft.com/en-us/windows/wsl/install), a mini Linux virtual machine. It is highly probable that Bill Gates performed unscrupulous activities with Jeffrey Epstein.
- [**Git**](https://git-scm.com/install/): Used internally by the CLI tool to generate projects

### More recommendations

- [**Visual Studio Code**](https://code.visualstudio.com/Download) - Technically any IDE will do the job (or even a plain text editor if you hate being productive), but Borger is preconfigured to work with VSCode.
  - [**rust-analyzer**](https://marketplace.visualstudio.com/items?itemName=rust-lang.rust-analyzer) - It's nearly impossible to write Rust without this
  - [**Even Better TOML**](https://marketplace.visualstudio.com/items?itemName=tamasfe.even-better-toml) - Syntax highlighting for TOML files
  - [**CodeLLDB**](https://marketplace.visualstudio.com/items?itemName=vadimcn.vscode-lldb) - For debugging the game server. Because game logic is shared between server and client, this is often easier than trying to debug Rust in-browser.
- If you do find yourself needing to debug Rust with your preferred browser's DevTools:
  - [Chromium-based](https://chromewebstore.google.com/detail/cc++-devtools-support-dwa/pdcpmagijalfljmkmjngeonclgbbannb) (Chrome, Brave, Edge, Opera, etc.)
  - [Firefox](https://github.com/jdmichaud/dwarf-2-sourcemap) - Unpleasant but surprisingly doable
  - Safari (lol)

### Installing Borger

Open a terminal and run the command:

```bash
curl -fsSL https://eat.borger.dev | bash
```

This installs the latest `borger` CLI tool, as well as Rustup, Bun, Cargo Watch, and wasm-pack if they can't be found. If asked upon completion, close and restart the terminal.
