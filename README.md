# Bazel + IntelliJ + Rust Example

Working example of Bazel + IntelliJ + Rust integration, allowing for all of IntelliJ's Rust code completion, navigation and refactoring features to work in a Bazel project.

https://user-images.githubusercontent.com/14253102/151391801-a7551fcb-0322-4ed5-adc1-f11784a3b106.mov

## Architecture

This integration is achieved using forks of the _Bazel_ and _Rust_ IntelliJ plugins:

- https://github.com/vaticle/bazel-intellij
- https://github.com/vaticle/intellij-rust

### Detecting project structure

The Rust plugin is built to work with Cargo projects, so to make it work with Bazel projects, we need to generate the Cargo manifest files using Bazel.

The modified Bazel plugin detects `rust_binary` and `rust_library` targets, and its `intellij_info_aspect` generates Cargo.toml files in the Bazel sandbox based on the `CrateInfo` of these targets.

The Rust plugin now sees `Cargo.toml` files in `bazel-bin` with `path` attributes such as `main.rs`. This allows `cargo` to detect project structure with `bazel-bin` as its root directory. But the source file paths that it detects are incorrect - the sources aren't in `bazel-bin`. So it maps the `path` attributes, from Bazel sandbox locations, to locations relative to the project root. This causes the plugin to treat the source folders as if they have Cargo.toml files in them with the correct properties, enabling code completion and navigation within the project.

`rust_test` targets need to be handled a little differently. A Bazel aspect can't modify the `Cargo.toml` generated by, say, a `rust_library`, while processing a `rust_test`. So to get IntelliJ to detect the test target, we write a _blank file_ to `bazel-bin` with the same name and path as the `rust_test`'s source file.

External crates from https://crates.io are supported using [cargo-raze](https://github.com/google/cargo-raze) - see https://github.com/vaticle/bazel-intellij-rust-example/tree/master/dependencies/crates for a usage example.

### Loading the Rust toolchain

The Rust plugin is able to detect the Rust toolchain in the Bazel sandbox, as well as the standard library sources.

### Current??limitations

Currently the plugins have only been tested against IntelliJ IDEA 2021.3 UE. Further changes may be needed to ensure compatibility with other environments.

The Rust toolchain paths are largely hardcoded - for example, the standard library is loaded from "bazel-${projectName}/external/rust_darwin_x64_amd/lib/" - this obviously only works on MacOS and is not the right way to go about finding the Rust toolchain. A correct solution would somehow communicate the path information from Bazel to the Rust plugin, perhaps using the `xxx-intellij-info.txt` files generated by `intellij_info_aspect`.

The overall architecture of the Bazel support in `intellij-rust` is suboptimal, and warrants a proper discussion.

## Contributing

We need support to bring these plugins to production quality, via any of the following means:

- Submit a PR to https://github.com/vaticle/bazel-intellij or https://github.com/vaticle/intellij-rust;
- We use Discord as our primary communications channel: https://vaticle.com/discord

