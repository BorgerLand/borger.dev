# Installation

### Hardware Requirements

Although games built with Borger are very small and efficient, running 2 `rustc` instances and `rust-analyzer` at the same time during development is absolutely brutal. At bare minimum, you will need at least 10GB of available disk storage and 12GB of RAM if using `rust-analyzer`.

Borger itself was written on a 2014 Dell Precision M2800 equipped with [Xubuntu](https://xubuntu.org/download/). If that clunker can run it, so can you!

### Software Requirements

- [**Bash**](https://www.gnu.org/software/bash/): Required to run Borger's CLI tool. Installing it depends on your operating system:
  - Linux: Any self-respecting distribution should already have it.
  - macOS: Requires [purchasing](https://www.apple.com/mac/) expensive, unrepairable, unupgradable hardware, but comes with Bash 3.2 (released in 2006) out of the box.
  - Windows: Requires [Windows Subsystem for Linux (WSL)](https://learn.microsoft.com/en-us/windows/wsl/install), a mini Linux virtual machine. High risk of Windows Update forcing a reboot that deletes your unsaved work. It is also highly probable that Bill Gates performed unscrupulous activities with Jeffrey Epstein.
- [**Git**](https://git-scm.com/install/): Used internally by the CLI tool to generate projects

### More recommendations

- [**Visual Studio Code**](https://code.visualstudio.com/Download) - Technically any IDE will do the job (or even a plain text editor if you're severely RAM-constrained), but Borger is preconfigured to work with VSCode.
  - [**rust-analyzer**](https://marketplace.visualstudio.com/items?itemName=rust-lang.rust-analyzer) - Makes an enormous difference in how easy it is to write Rust
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

This installs the latest `borger` CLI tool, as well as Rustup, Bun, Cargo Watch, and wasm-pack if they can't be found. If asked upon completion, close and restart the terminal. Do note the security implications of downloading and installing several programs from the internet.
