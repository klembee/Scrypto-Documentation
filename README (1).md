# Installation

### Install Rust

#### Windows

1. Download and install [`rustup-init.exe`](https://win.rustup.rs/x86\_64)
2. Install "Desktop development with C++" with [Build Tools for Visual Studio 2019](https://visualstudio.microsoft.com/thank-you-downloading-visual-studio/?sku=BuildTools\&rel=16)
3. Start a new PowerShell to enable cargo

#### Linux and

1.  Install rust:

    ```
    curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
    ```
2.  Enable cargo in the current shell:

    ```
    source $HOME/.cargo/env
    ```

### Setup Scrypto simulator

1.  Add WebAssembly target

    ```
    rustup target add wasm32-unknown-unknown
    ```
2.  Install the simulator

    ```
    git clone git@github.com:radix-scrypto/peek-scrypto.git
    cd peek-scrypto
    cargo install --path ./simulator
    ```





###

###
