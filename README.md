This is an informal stress test for the Rust language and toolchain using [Rust/WinRT](https://github.com/microsoft/winrt-rs). It highlights how much slower the Rust compiler is for this particular use case compared with modern C++ compilers. Hopefully, this can help folks working on the Rust compiler and rust-analyzer frontend.

The bindings crate produces around 160MB of Rust source code - about 3.2 million lines of code. On my machine, a completely fresh build (i.e., `cargo build`) takes around 15 minutes to compile. The comparable C++ code from [C++/WinRT](https://github.com/microsoft/cppwinrt) takes about 90 seconds to compile.

My machine's specs are roughly as follows:
* Intel i7-9700 3 GHz CPU 
* 32 GB of RAM
* Running Windows 10 on WSL

## Build
```
cargo install cargo-winrt --path winrt-rs/crates/cargo-winrt
cargo winrt install 
cargo build
```

## Initial Investigation

Initial investigation into why this compilation is slow has shown that the heavy use of `From` trait impls to mimic WinRT's OOP interface leads to massive slow downs in coherence testing. An in-depth look at this initial investigation [can be found here](https://gist.github.com/lqd/88119aea556fb90cc9824a5dd4e816df).

## Investigation

### Is rustc or LLVM the cause of the majority of the slow down?

On a clean build `cargo build` seems to take around the 19m50s time frame while `cargo check` takes 17m00s. This indicates that it is likely rustc, not LLVM which is producing the bulk of the slow down though LLVM does seem to be slower than we would expect.
