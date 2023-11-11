# Android

### Quick links
* [Overview](#overview)

## Research
* Cargo NDK
* Cargo mobile
* Cargo ndk

**Resources**
* [Flapigen](https://github.com/Dushistov/flapigen-rs)
* [Android source with Rust](https://source.android.com/docs/setup/build/rust/building-rust-modules/overview)

* 99.9% of the code is in Rust/wasm32
* iOS/Android just provide a minimal "shell" for loading a webview + providing some bindings to native APIs

1. create a webview
2. load wasm in the webview
3. setup native <-> webview <-> rust/wasm communication


<!-- 
vim: ts=2:sw=2:sts=2
-->
