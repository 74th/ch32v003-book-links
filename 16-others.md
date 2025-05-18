# 16. その他の機能

## NRST、SWIOピンを使用する

```sh
# PD7をGPIOとして使う
minichlink -D
# PD7をNRSTとして使う
minichlink -d
```

## ユーザ情報を格納するOption Byte

> https://github.com/cnlohr/ch32fun/blob/master/examples/optionbytes/optionbytes.c

## Rustでの開発環境の構築

```sh
rustup install nightly
```

> https://github.com/ch32-rs/ch32-hal-template

```sh
cargo install cargo-generate
```

```sh
cargo generate ch32-rs/ch32-hal-template

🤷   Project Name: test-template
🔧   Destination: ...
🔧   project-name: test-template ...
🔧   Generating template ...
✔ 🤷   Enable embassy (async) support? · false
✔ 🤷   Which MCU family to target? · ch32v003
✔ 🤷   Which ch32v003 variant to use? · ch32v003f4p6
🔧   Initializing a fresh Git repository
✨   Done! New project created ...
```

```toml
[dependencies]
ch32-hal = { git = "https://github.com/ch32-rs/ch32-hal.git",
  default-features = false,
  features = [
    "ch32v003f4u6",
    "time-driver-tim2",
    # "rt",
    "memory-x",
] }
```

```sh
cargo run
```

```rust
#[cfg(debug_assertions)]
hal::debug::SDIPrint::enable();

#[cfg(debug_assertions)]
hal::println!("hello world");
```

```sh
cargo build --release
```
