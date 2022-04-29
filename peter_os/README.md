# Outline of OS   

## Part 1: Barebone

**Core: Remove all OS Dependent Functions**


1. ```#![no_std]```: inner attribute
2. panic define (<- remain ```eh_persionality``` error)
3. ```eh_persionality```: Stack-unwinding Mark. (-> "abort panic" to disable)
4. error: requires 'start' lang item

That is because ```runtime system``` could not be initailized.

in rust binary links std, exe starts in a C runtime lib called ```crt0```, which includes creating a stack and placing the arguments in the right registers. The C runtime then invokes the entry point of the Rust runtime, which is marked by the ```start``` language item.

impl 'start' is useless, so overwrite ```crt0```.

5. Overwriting thee Entry Point: ```#![no_main]``` and rewrite '_start()'

```rust
#![no_main] 
#[no_mangle]
pub extern "C" fn _start() {

}
```

6. Linker error

The linker is a program that combines the generated code into an executable.

*Solution*: 

- Tell linker that it should not include C runtime.

Use a bare metal enviroment: ```thumbv7em-none-eabihf```

Actually twhich env we use is not such important, all that matters is that the target triple has no underlying OS.

```shell
$ rustup target add thumbv7em-none-eabihf

$ cargo build --target thumbv7em-none-eabihf
```

Thus we cross compile our exe for bare metal target system, because the target system has no operating system, the linker does not try to link the C runtime and our build succeeds without any linker errors.

- Pass a certain set of arguments to the linker or by building for a bare metal target.

```shell
$ cargo rustc -- -C link-args="-e __start -static -nostartfiles"
```

"-e __start": specify '_start' entry.

"-static": macOS does not officailly support statically linked binaries, so we pass "-static" flag.

"-nostartfiles": prevent to link to "crt0"

## Part 2: A Minimal Rust Kernel

**Bootloader**

1. Always use BIOS & UEFI firmware.

2. Bootloader: Determine the location of the kernel image on the disk and load it into memory.

```16-bit real mode``` --> ```32-bit protected mode``` --> ```64-bit long mode```

3. Use ```bootimag```e to instead cumbersome bootloader.

4. Setup ```nightly``` mode.

New file "rust-toolchain" which contains "nightly".

5. Set target triple: "x86_64-peter_os.json" and set ```.cargo/config.toml```

6. release ```mem-related``` and print strings in screen.

7. Create ```disk-image``` and link bootloader by adding dependency and ```cargo bootimage```.

8. Run on QEMU VM.

## Part 3: VGA Text Mode