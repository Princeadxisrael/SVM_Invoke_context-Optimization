<p align="center">
  <a href="https://solana.com">
    <img alt="Solana" src="https://i.imgur.com/IKyzQ6T.png" width="250" />
  </a>
</p>

[![Solana crate](https://img.shields.io/crates/v/solana-core.svg)](https://crates.io/crates/solana-core)
[![Solana documentation](https://docs.rs/solana-core/badge.svg)](https://docs.rs/solana-core)
[![Build status](https://badge.buildkite.com/8cc350de251d61483db98bdfc895b9ea0ac8ffa4a32ee850ed.svg?branch=master)](https://buildkite.com/solana-labs/solana/builds?branch=master)
[![codecov](https://codecov.io/gh/solana-labs/solana/branch/master/graph/badge.svg)](https://codecov.io/gh/solana-labs/solana)


# Project

A review, annotation and optimization of the Solana core infrastructure including the TPU, Validator processes, Runtime, Consesus mechanisms and the Runtime Invoke_Context with particular focus on how the Runtime uses Rust to  consume Programs, Accounts, Instructions, and Data. 


## Optimizations

Check out the optimization chart: [SVM optimization chart](https://docs.google.com/document/d/1f4VHvumYd2pNMWijQUqWre7i86ximymFHrxTggl9iGQ/edit?usp=sharing)

1. The prepare_instruction method in the invoke_context module (line 386-394) was optimized by using a cloned() method:


```bash
let instruction_accounts = duplicate_indicies
.into_iter().map(|duplicate_index| {deduplicated_instruction_accounts
.get(duplicate_index)
.cloned()  // Use .cloned() here
.ok_or(InstructionError::NotEnoughAccountKeys)
})
.collect::<Result<Vec<InstructionAccount>, InstructionError>>()?;
```
While the functionality will remain the same, the efficiency of the program is improved by use of .cloned() which simplifies the logic because it applies the cloning operation on the Option<T> directly thus avoiding the need for an additional ok_or() call, reducing complexity and the number of intermediate operations. It's slightly more concise and can be marginally faster because it avoids an additional OK() wrapping step.



# Building

## **1. Install rustc, cargo and rustfmt.**

```bash
$ curl https://sh.rustup.rs -sSf | sh
$ source $HOME/.cargo/env
$ rustup component add rustfmt
```

When building the master branch, please make sure you are using the latest stable rust version by running:

```bash
$ rustup update
```

When building a specific release branch, you should check the rust version in `ci/rust-version.sh` and if necessary, install that version by running:
```bash
$ rustup install VERSION
```
Note that if this is not the latest rust version on your machine, cargo commands may require an [override](https://rust-lang.github.io/rustup/overrides.html) in order to use the correct version.

On Linux systems you may need to install libssl-dev, pkg-config, zlib1g-dev, protobuf etc.

On Ubuntu:
```bash
$ sudo apt-get update
$ sudo apt-get install libssl-dev libudev-dev pkg-config zlib1g-dev llvm clang cmake make libprotobuf-dev protobuf-compiler
```

On Fedora:
```bash
$ sudo dnf install openssl-devel systemd-devel pkg-config zlib-devel llvm clang cmake make protobuf-devel protobuf-compiler perl-core
```

## **2. Download the source code.**

```bash
$ git clone https://github.com/solana-labs/solana.git
$ cd solana
```

## **3. Build.**

```bash
$ ./cargo build
```

# Testing

**Run the test suite:**

```bash
$ ./cargo test
```

### Starting a local testnet

Start your own testnet locally, instructions are in the [online docs](https://docs.solanalabs.com/clusters/benchmark).

### Accessing the remote development cluster

* `devnet` - stable public cluster for development accessible via
devnet.solana.com. Runs 24/7. Learn more about the [public clusters](https://docs.solanalabs.com/clusters)

# Benchmarking

First, install the nightly build of rustc. `cargo bench` requires the use of the
unstable features only available in the nightly build.

```bash
$ rustup install nightly
```

Run the benchmarks:

```bash
$ cargo +nightly bench
```

# Release Process

The release process for this project is described [here](RELEASE.md).

# Code coverage

To generate code coverage statistics:

```bash
$ scripts/coverage.sh
$ open target/cov/lcov-local/index.html
```
