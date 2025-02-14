# Lab 0: Environment Setup

## 1. Cross-Platform Development

**Cross Compiler**

- **GNU**
  - To install.
    ```bash
    brew tap messense/macos-cross-toolchains
    brew install aarch64-unknown-linux-gnu
    aarch64-linux-gnu-gcc --version
    ```
  - To check path.
    ```bash
    which aarch64-linux-gnu-gcc
    ```
  - Create a file named env.gnu with the following content:
    ```bash
    export PATH="/path/to/gnu/toolchain/bin:$PATH"
    ```
  - To activate.
    ```bash
    source env.gnu
    ```    
- **LLVM (includes clang and ld.lld)**
  - To install.
    ```bash
    brew install llvm
    clang --version
    brew install lld
    ld.lld --version
    ```
  - To check path.
    ```bash
    brew info llvm
    ```
  - Create a file named env.llvm with the following content:
    ```bash
    export PATH="/path/to/llvm/bin:$PATH"
    export LDFLAGS="-L/path/to/llvm/lib"
    export CPPFLAGS="-I/path/to/llvm/include"
    ```
  - To activate.
    ```bash
    source env.llvm
    ```

**Linker**

- Create a file named linker.ld with the following content:
  ```
  SECTIONS
  {
    . = 0x80000;
    .text : { *(.text) }
  }
  ```

**QEMU**

- To install.
  ```bash
  brew install qemu
  qemu-system-aarch64 --version
  ```
  
## 2. From Source Code to Kernel Image

**From Source Code to Object Files**

- Create a file named a.S with the following content:
  ```
  .section ".text"
  _start:
    wfe
    b _start
  ```
- Compile a.S to a.o.
  ```bash
  # GNU
  aarch64-linux-gnu-gcc -c a.S
  # LLVM
  clang -mcpu=cortex-a53 --target=aarch64-rpi3-elf -c a.S
  ```

- **aarch64-linux-gnu-gcc**  
  - Targets the AArch64 architecture on Linux.
  - Generates code for Linux systems without specifying a particular CPU.

- **clang -mcpu=cortex-a53 --target=aarch64-rpi3-elf**  
  - Explicitly specifies the target CPU as Cortex-A53.
  - Sets the target to a bare-metal environment for Raspberry Pi 3.

**From Object Files to ELF**

- Link a.o to kernel8.elf.
  ```bash
  # GNU
  aarch64-linux-gnu-ld -T linker.ld -o kernel8.elf a.o
  # LLVM
  ld.lld -m aarch64elf -T linker.ld -o kernel8.elf a.o
  ```

**From ELF to Kernel Image**

- Convert kernel8.elf to kernel8.img.
  ```bash
  # GNU
  aarch64-linux-gnu-objcopy -O binary kernel8.elf kernel8.img
  # LLVM
  llvm-objcopy --output-target=aarch64-rpi3-elf -O binary kernel8.elf kernle8.img
  ```
  
**Check on QEMU**
```bash
qemu-system-aarch64 -M raspi3b -kernel kernel8.img -display none -d in_asm
```
