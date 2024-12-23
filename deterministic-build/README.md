# Deterministic Build

This is a Docker image to produce reproducible builds of Dusk smart contracts.

To build the image:

```bash
cd deterministic-build
docker build --platform linux/amd64 . -t dusk-reproducible-build
```

The platform is specified as `linux/amd64` because an arm image will not generate the
same output.

Usage:

```bash
docker run --rm \
    -v <path-to-contract-code>:/source \
    -v <path-to-output-folder>:/target \
    --mount type=volume,source=dusk_registry_cache,target=/root/.cargo/registry \
    dusk-reproducible-build
```

Replace `<path-to-contract-code>` and `<path-to-output-folder>` with the path to your smart contract's project
and the path to the folder where you want the output to be, respectively.
After successful compilation, all build artifacts will be in `<path-to-output-folder>` and the final reproducible outputs
will be in `<path-to-output-folder>/final-output/wasm32` and `<path-to-output-folder>/final-output/wasm64`, depending on
the specified target.

The container will run a cargo build, then strip the output binary using wasm-tools.
By default, cargo build is invoked with arguments `--locked --color=always --release --target wasm32-unknown-unknown`
and `RUSTFLAGS` set to `-C link-args=-zstack-size=65536`.
To override the default `--target` argument, you can specify:

```bash
docker run --rm \
    -v <path-to-contract-code>:/source \
    -v <path-to-output-folder>:/target \
    --mount type=volume,source=dusk_registry_cache,target=/root/.cargo/registry \
    dusk-reproducible-build
    --target wasm64-unknown-unknown
```

And this will run `cargo build` with arguments `--locked --color=always --release --target wasm64-unknown-unknown`
Any argument you specify in this manner will override the default `--target wasm32-unknown-unknown` but the others
`--locked --color=always --release` will still remain. For example:

```bash
docker run --rm \
    -v <path-to-contract-code>:/source \
    -v <path-to-output-folder>:/target \
    --mount type=volume,source=dusk_registry_cache,target=/root/.cargo/registry \
    dusk-reproducible-build
    --manifest-path contracts/charlie/Cargo.toml
```

This will run `cargo build --locked --color=always --release --manifest-path contracts/charlie/Cargo.toml`, but the
wasm target won't be specified.
