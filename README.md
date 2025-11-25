# libbpf-cross-compile

This repo was created to help test changes to the libbpf-sys library to help it support cross-compilation (for Android in my case).

Tested on ubuntu:24.04 docker image.


## Setup

* Initialize the libelf and libbpf submodules with `git submodule update --init --recursive`

* Install Rust using rustup

* Add Android build target with `rustup target add aarch64-linux-android`

* Download Android's NDK (I use r26d)

* Install `m4 make autotools-dev autoconf gettext autopoint flex bison clang gcc-multilib gawk clang-format pkg-config libelf-dev`

* Set the envrionment variable `ANDROID_NDK_ROOT` to point to the extracted NDK (`.../android-ndk-r26d`).

    You may skip this step and pass `ANDROID_NDK_ROOT=...` to `make` instead.

* Run `make` to build libelf and libbpf, and configure cargo with the correct toolchain.

    You should only have to call `make` once (unless updating the submodules). From that point, use `cargo` to build the project.

## libbpf-sys Cross compilation

The `Makefile` in this project generates a `.cargo/config.toml` that specifies the `LIBBPF_SYS_LIBRARY_PATH_aarch64_linux_android` and `LIBBPF_SYS_LIBRARY_PATH_x86_64_unknown_linux_gnu` variables. If we try to use only the old `LIBBPF_SYS_LIBRARY_PATH` variable we would get:

```
root@38490824fbe8:/libbpf-cross-compile# cargo build
   Compiling libbpf-sys v1.6.2+v1.6.2 (https://github.com/amirbou/libbpf-sys.git?branch=per-target-sys-library-path#12471592)
   Compiling libbpf-rs v0.26.0-beta.1 (https://github.com/amirbou/libbpf-rs.git#5aa8d118)
   Compiling libbpf-cargo v0.26.0-beta.1 (https://github.com/amirbou/libbpf-rs.git#5aa8d118)
   Compiling libbpf-cross-compile v0.1.0 (/libbpf-cross-compile)
error: linking with `cc` failed: exit status: 1
  |
  = note:  "cc" "-m64" "/tmp/rustcYpmfUd/symbols.o" "<6 object files omitted>" "-Wl,--as-needed" "-Wl,-Bstatic" "<sysroot>/lib/rustlib/x86_64-unknown-linux-gnu/lib/{libstd-*,libpanic_unwind-*,libobject-*,libmemchr-*,libaddr2line-*,libgimli-*,libcfg_if-*,librustc_demangle-*,libstd_detect-*,libhashbrown-*,librustc_std_workspace_alloc-*,libminiz_oxide-*,libadler2-*,libunwind-*,liblibc-*,librustc_std_workspace_core-*,liballoc-*,libcore-*,libcompiler_builtins-*}.rlib" "-Wl,-Bdynamic" "-lgcc_s" "-lutil" "-lrt" "-lpthread" "-lm" "-ldl" "-lc" "-L" "/tmp/rustcYpmfUd/raw-dylibs" "-B<sysroot>/lib/rustlib/x86_64-unknown-linux-gnu/bin/gcc-ld" "-fuse-ld=lld" "-Wl,--eh-frame-hdr" "-Wl,-z,noexecstack" "-L" "/libbpf-cross-compile/target/debug/build/libbpf-sys-81db787185a5d4c6/out" "-L" "/libbpf-cross-compile/target/debug/build/vsprintf-d632bc991125b981/out" "-L" "/libbpf-cross-compile//static_libs/aarch64-linux-android/lib" "-L" "/android-ndk-r26d/toolchains/llvm/prebuilt/linux-x86_64/bin/../sysroot/usr/lib/aarch64-linux-android/30" "-L" "/android-ndk-r26d/toolchains/llvm/prebuilt/linux-x86_64/bin/../sysroot/usr/lib/aarch64-linux-android/" "-L" "<sysroot>/lib/rustlib/x86_64-unknown-linux-gnu/lib" "-o" "/libbpf-cross-compile/target/debug/build/libbpf-cross-compile-a6ca8222de2eeae5/build_script_build-a6ca8222de2eeae5" "-Wl,--gc-sections" "-pie" "-Wl,-z,relro,-z,now" "-nodefaultlibs"
  = note: some arguments are omitted. use `--verbose` to show all linker arguments
  = note: rust-lld: error: /android-ndk-r26d/toolchains/llvm/prebuilt/linux-x86_64/bin/../sysroot/usr/lib/aarch64-linux-android/30/libm.so is incompatible with elf_x86_64
          rust-lld: error: /android-ndk-r26d/toolchains/llvm/prebuilt/linux-x86_64/bin/../sysroot/usr/lib/aarch64-linux-android/30/libdl.so is incompatible with elf_x86_64
          rust-lld: error: /android-ndk-r26d/toolchains/llvm/prebuilt/linux-x86_64/bin/../sysroot/usr/lib/aarch64-linux-android/30/libc.so is incompatible with elf_x86_64
          collect2: error: ld returned 1 exit status


error: could not compile `libbpf-cross-compile` (build script) due to 1 previous error
```