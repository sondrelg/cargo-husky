Husky for Cargo :dog:
=====================
[![Crates.io][crates-io badge]][cargo-husky]
[![Build Status on Linux/macOS][travis-ci badge]][travis-ci]
[![Build status on Windows][appveyor badge]][appveyor]

[cargo-husky][] is a crate for Rust project managed by [cargo][]. In short, cargo-husky is a Rust
version of [husky][].

cargo-husky is a development tool to set Git hooks automatically on `cargo test`. By hooking `pre-push`
and running `cargo test` automatically, it prevents broken codes from being pushed to a remote
repository.


## Usage

Please add `cargo-husky` crate to `[dev-dependencies]` section of your project's `Cargo.toml`.

```toml
[dev-dependencies]
cargo-husky = "1"
```

Then run tests in your project directory.

```
$ cargo test
```

Check Git hook was generated at `.git/hooks/pre-push`.
cargo-husky generates a hook script which runs `cargo test` by default.

e.g.

```bash
#!/bin/sh
#
# This hook was set by cargo-husky v1.0.0: https://github.com/rhysd/cargo-husky#readme
# Generated by script /path/to/cargo-husky/build.rs
# Output at /path/to/target/debug/build/cargo-husky-xxxxxx/out
#

set -e

echo '+cargo test'
cargo test
```

Note: cargo-husky does nothing on `cargo test` when
- hook script was already generated by the same version of cargo-husky
- another hook script put by someone else is already there

To uninstall cargo-husky, please remove `cargo-husky` from your `[dev-dependencies]` and remove
hook scripts from `.git/hooks`.

[Japanese blogpost](https://rhysd.hatenablog.com/entry/2018/10/08/205041)


## Customize behavior

Behavior of cargo-husky can be customized by feature flags of `cargo-husky` package.
You can specify them in `[dev-dependencies.cargo-husky]` section of `Cargo.toml` instead of adding
`cargo-husky` to `[dev-dependencies]` section.

e.g.

```toml
[dev-dependencies.cargo-husky]
version = "1"
default-features = false # Disable features which are enabled by default
features = ["precommit-hook", "run-cargo-test", "run-cargo-clippy"]
```

This configuration generates `.git/hooks/pre-commit` script which runs `cargo test` and `cargo clippy`.

All features are follows:

| Feature            | Description                                                         | Default  |
|--------------------|---------------------------------------------------------------------|----------|
| `run-for-all`      | Add `--all` option to command to run it for all crates in workspace | Enabled  |
| `prepush-hook`     | Generate `pre-push` hook script                                     | Enabled  |
| `precommit-hook`   | Generate `pre-commit` hook script                                   | Disabled |
| `postmerge-hook`   | Generate `post-merge` hook script                                   | Disabled |
| `run-cargo-test`   | Run `cargo test` in hook scripts                                    | Enabled  |
| `run-cargo-check`  | Run `cargo check` in hook scripts                                   | Disabled |
| `run-cargo-clippy` | Run `cargo clippy -- -D warnings` in hook scripts                   | Disabled |
| `run-cargo-fmt`    | Run `cargo fmt -- --check` in hook scripts                          | Disabled |
| `user-hooks`       | See below section                                                   | Disabled |


## User Hooks

If generated hooks by `run-cargo-test` or `run-cargo-clippy` features are not sufficient for you,
you can create your own hook scripts and tell cargo-husky to put them into `.git/hooks` directory.

1. Create `.cargo-husky/hooks` directory at the same directory where `.git` directory is put.
2. Create hook files such as `pre-push`, `pre-commit`, ... as you like.
3. Give an executable permission to the files (on \*nix OS).
4. Write `features = ["user-hooks"]` to `[dev-dependencies.cargo-husky]` section of your `Cargo.toml`.
5. Check whether it works by removing an existing `target` directory and run `cargo test`.

e.g.

```
your-repository/
├── .git
└── .cargo-husky
    └── hooks
        ├── post-merge
        └── pre-commit
```

```toml
[dev-dependencies.cargo-husky]
version = "1"
default-features = false
features = ["user-hooks"]
```

cargo-husky inserts an information header to copied hook files in `.git/hooks/` in order to detect
self version update.

Note that, when `user-hooks` feature is enabled, all other features are disabled. You need to prepare
all hooks in `.cargo-husky/hooks` directory.


## Ignore Installing Hooks

When you don't want to install hooks for some reason, please set `$CARGO_HUSKY_DONT_INSTALL_HOOKS`
environment variable.

```
CARGO_HUSKY_DONT_INSTALL_HOOKS=true cargo test
```


## How It Works

[husky][] utilizes npm's hook scripts, but cargo does not provide such hooks.
Instead, cargo-husky sets Git hook automatically on running tests by [cargo's build script feature][build scripts].

Build scripts are intended to be used for building third-party non-Rust code such as C libraries.
They are automatically run on compiling crates.

If `cargo-husky` crate is added to `dev-dependencies` section, it is compiled at running tests.
At the timing, [build script](./build.rs) is run and sets Git hook automatically.
The build script find the `.git` directory to put hooks based on `$OUT_DIR` environment variable
which is automatically set by `cargo`.

cargo-husky puts Git hook file only once for the same version. When it is updated to a new version,
it overwrites the existing hook by detecting itself was updated.

cargo-husky is developed on macOS and tested on Linux/macOS/Windows with 'stable' channel Rust toolchain.

## License

[MIT](./LICENSE.txt)

[cargo-husky]: https://crates.io/crates/cargo-husky
[cargo]: https://github.com/rust-lang/cargo
[husky]: https://github.com/typicode/husky
[build scripts]: https://doc.rust-lang.org/cargo/reference/build-scripts.html
[travis-ci badge]: https://travis-ci.org/rhysd/cargo-husky.svg?branch=master
[travis-ci]: https://travis-ci.org/rhysd/cargo-husky
[appveyor badge]: https://ci.appveyor.com/api/projects/status/whby8hq44tf9bob4/branch/master?svg=true
[appveyor]: https://ci.appveyor.com/project/rhysd/cargo-husky/branch/master
[crates-io badge]: https://img.shields.io/crates/v/cargo-husky.svg
