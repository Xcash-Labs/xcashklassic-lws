# xcashklassic-lws

`xcashklassic-lws` is an XCash Klassic adaptation of `monero-lws`, a light wallet server implementation based on the Monero light-wallet REST API.

This project is separate from `xcash-labs-core`. It uses `xcash-labs-core` as a build dependency through the `external/monero` submodule path. The folder name is kept for compatibility with the upstream `monero-lws` CMake layout, but the submodule should point to:

```text
https://github.com/Xcash-Labs/xcash-labs-core.git
```

## Table of Contents

- [Introduction](#introduction)
- [About this project](#about-this-project)
- [Repository layout](#repository-layout)
- [Dependencies](#dependencies)
- [Build instructions](#build-instructions)
- [Running xcashklassic-lws](#running-xcashklassic-lws)
- [License](#license)

## Introduction

XCash Klassic is a CryptoNote-based blockchain derived from Monero technology. This repository provides a light wallet server layer for XCash Klassic so compatible wallets or services can submit wallet view information and have the server scan the blockchain for wallet activity.

The goal is to provide an XCash Klassic-compatible light wallet server while preserving as much of the upstream `monero-lws` structure as possible.

## About this project

This project is based on `monero-lws`, which implements the Monero light-wallet REST API. The original project supports MyMonero-style clients and scans blockchain data for wallets whose view keys are registered with the server.

Key characteristics inherited from upstream `monero-lws` include:

- LMDB-backed storage
- View keys stored in the database
- Continuous background scanning
- ZeroMQ daemon integration for chain subscription support
- Optional webhook notifications
- AMD64 ASM acceleration from the core project when available

For XCash Klassic, the important build change is that `external/monero` should contain `xcash-labs-core`, not upstream Monero.

## Repository layout

Expected layout:

```text
xcashklassic-lws/
  CMakeLists.txt
  src/
  external/
    monero/        # XCash Labs Core submodule, kept at this path for compatibility
  build/           # xcashklassic-lws build output
```

The `external/monero` path name is intentional. It avoids large CMake changes in the LWS project while allowing the dependency to be replaced with XCash Labs Core.

## Dependencies

Install the normal XCash Labs Core / Monero-style build dependencies first. `xcashklassic-lws` depends on the core project for headers, static libraries, daemon RPC/ZMQ types, LMDB support, crypto code, and related build targets.

No additional special dependency is expected beyond what is needed to build `xcash-labs-core` and `monero-lws`.

## Build instructions

These instructions build the XCash Labs Core submodule separately first, then build `xcashklassic-lws` against that completed core build.

This avoids pulling the core project into the LWS CMake process as a nested project, which can cause CMake source-directory issues.

### 1. Clone the repository

```bash
git clone https://github.com/Xcash-Labs/xcashklassic-lws.git
cd xcashklassic-lws
```

### 2. Initialize submodules

```bash
git submodule update --init --recursive
```

Verify that `external/monero` points to XCash Labs Core:

```bash
cd external/monero
git remote -v
cd ../..
```

You should see:

```text
https://github.com/Xcash-Labs/xcash-labs-core.git
```

### 3. Build XCash Labs Core first

From the `xcashklassic-lws` repository root:

```bash
cd external/monero

mkdir -p build/Linux/master/release
cd build/Linux/master/release

cmake -DCMAKE_BUILD_TYPE=Release ../../../..
make -j$(nproc) daemon multisig lmdb_lib
```

This creates the core build directory used by LWS:

```text
external/monero/build/Linux/master/release
```

### 4. Build xcashklassic-lws

Return to the `xcashklassic-lws` repository root:

```bash
cd ~/xcashklassic-lws
```

If your repository is somewhere else, replace the path with your actual location.

Then build LWS:

```bash
rm -rf build
mkdir build
cd build

cmake -DCMAKE_BUILD_TYPE=Release \
  -DMONERO_SOURCE_DIR=$HOME/xcashklassic-lws/external/monero \
  -DMONERO_BUILD_DIR=$HOME/xcashklassic-lws/external/monero/build/Linux/master/release \
  ..

make -j$(nproc)
```

The resulting executables should be placed under:

```text
build/src
```

## Running xcashklassic-lws

The build places the daemon binary in the `src/` subdirectory inside the LWS build directory.

From the LWS build directory:

```bash
./src/monero-lws-daemon --help
```

The binary may still use the upstream `monero-lws-daemon` name until it is renamed in the project.

At runtime, `xcashklassic-lws` needs to connect to a running XCash Klassic daemon. Building against a local `external/monero` source tree does not start a second node. A typical runtime layout is:

```text
xcashd                  # running XCash Klassic daemon
xcashklassic-lws daemon # connects to xcashd over RPC/ZMQ
wallet clients          # connect to the LWS service
```

Check available runtime options with:

```bash
./src/monero-lws-daemon --help
```

## Notes for maintainers

If the `external/monero` submodule needs to be reset to XCash Labs Core master:

```bash
git submodule deinit -f external/monero
git rm -f external/monero
rm -rf .git/modules/external/monero

git submodule add -b master https://github.com/Xcash-Labs/xcash-labs-core.git external/monero
git submodule update --init --recursive
```

On Windows CMD, replace the `rm -rf` command with:

```cmd
rmdir /s /q .git\modules\external\monero
```

Then commit the submodule configuration and pointer:

```bash
git add .gitmodules external/monero
git commit -m "Use XCash Labs Core as LWS submodule"
```

## License

See [LICENSE](LICENSE).
