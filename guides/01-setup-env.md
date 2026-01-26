In order to work in this project there are some basics tools you need to have installed in your computer:

# Development Environment Setup

- [go](https://go.dev/doc/install) && `go env GOTOOLCHAIN`
- [cargo](https://www.rust-lang.org/tools/install)
- [foundryup](https://getfoundry.sh/)
- git
- docker
- nodejs I prefer using [nvm](https://github.com/nvm-sh/nvm)
- `cmake make gcc`

# Test Environment Setup

- `clang llvm lld`

```sh
cd /tmp
wget https://github.com/WebAssembly/wasi-sdk/releases/download/wasi-sdk-25/wasi-sdk-25.0-x86_64-linux.tar.gz
sudo tar xf wasi-sdk-25.0-x86_64-linux.tar.gz -C /opt
sudo ln -sf /opt/wasi-sdk-25.0-x86_64-linux /opt/wasi-sdk
rm wasi-sdk-25.0-x86_64-linux.tar.gz
```

```sh
# Rust WASM targets
rustup target add wasm32-unknown-unknown
rustup target add wasm32-wasip1
```

```sh
go install gotest.tools/gotestsum@latest
```
