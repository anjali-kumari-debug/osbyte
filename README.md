# Osbyte Operating System

A 64-bit ARM-based operating system designed for educational and research purposes, featuring a console-inspired graphical interface and comprehensive networking capabilities.

## Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Architecture](#architecture)
- [Project Structure](#project-structure)
- [Prerequisites](#prerequisites)
- [Building and Running](#building-and-running)
- [Platform Support](#platform-support)
- [Networking](#networking)
- [Development](#development)
- [License](#license)

## Overview

Osbyte is a custom-built 64-bit operating system targeting ARM processors, developed as an educational project to explore low-level systems programming, kernel development, and operating system design principles. The system can run in QEMU emulation environments (virt board or Raspberry Pi 4B emulation) and has preliminary support for physical Raspberry Pi hardware.

The operating system features a video game console-inspired graphical user interface that is fully keyboard-controllable, with plans for mouse and gamepad support. It implements a modular architecture with kernel processes and user-space applications, though current support for third-party processes is limited to self-contained programs.

**Note:** This is a hobby project and represents a first attempt at operating system development. The codebase is subject to continuous improvement and may not represent production-quality code.

## Features

- **64-bit ARM Architecture**: Full support for ARMv8-A instruction set
- **Multi-platform Support**: Runs on QEMU virt board, Raspberry Pi 4B/5 emulation, and physical hardware
- **Graphical Interface**: Console-inspired GUI with keyboard navigation
- **Networking Stack**: Comprehensive IPv4/IPv6 networking with TCP/UDP support
- **File System Support**: FAT32 and exFAT filesystem drivers
- **Process Management**: Kernel and user-space process support with scheduler
- **Device Drivers**: USB, audio, graphics, and storage device support
- **Development Tools**: Integrated debugging support via GDB

## Architecture

### Boot Process

The system boot sequence begins with `boot.S`, which initializes the stack pointer (defined in `linker.ld`) and transfers control to the `kernel_main` function in `kernel.c`. The linker script (`linker.ld`) defines the kernel's memory layout and load address.

### Kernel Initialization

The `kernel_main` function in `kernel.c` handles system initialization, including:
- Memory management setup (MMU, page allocator)
- Interrupt and exception handling
- Device driver initialization
- Process scheduler initialization
- Filesystem mounting
- Network stack initialization

After initialization, control is handed over to the process scheduler, which manages both kernel processes and user processes.

### Process Model

The system supports two types of processes:

1. **Kernel Processes**: Embedded directly in the kernel binary, placed in a dedicated linker section. These processes have direct access to kernel internals and are responsible for core system functionality.

2. **User Processes**: Loaded from the filesystem as ELF executables. These processes run in user space and interact with the kernel through system calls.

**Current Limitations**: Some initialization code (GPU and XHCI input) requires a process context, so a temporary kernel process is created during boot. Future improvements will modularize this architecture, potentially allowing kernel processes to be loaded from the filesystem.

The system can boot from both `kernel.elf` and `kernel.img` formats on virt and Raspberry Pi 4B platforms.

## Project Structure

The project is organized into three main components:

### Kernel (`kernel/`)

Contains the core operating system code, including:
- Boot loader (`boot.S`)
- Kernel entry point (`kernel.c`)
- Memory management (MMU, page allocator, DMA)
- Process management and scheduling
- System call interface
- Exception and interrupt handling
- Device drivers
- Filesystem implementations
- Network stack
- Kernel processes

**Key Files:**
- `boot.S`: Assembly boot code
- `kernel.c`: Main kernel initialization
- `linker.ld`: Memory layout definition
- `process/scheduler.c`: Process scheduler
- `process/syscall.c`: System call interface

### Shared Library (`shared/`)

A static library providing common functionality for both kernel and user processes:

- **Standard Library (`std/`)**: Minimal C standard library implementation
  - C++ `new` and `delete` operators
  - Fixed-size arrays and index maps
  - String manipulation functions
  - Memory operations (copy, clear, set)
  - Access via `#include "std/std.h"`

- **UI Library (`ui/`)**: Graphics and user interface primitives
  - 2D framebuffer drawing functions
  - Basic C++ Label class
  - Future expansion planned for additional UI components
  - Access via `#include "ui/ui.h"`

- **Math Library (`math/`)**: Mathematical utilities
  - Vector operations
  - AABB (Axis-Aligned Bounding Box) calculations
  - Random number generation
  - Access via `#include "math/math.h"`

- **Input Handling**: Keyboard input structures and keycode definitions
  - Access via `#include "input_keycodes.h"` and `#include "keyboard_input.h"`

### User Processes (`user/`)

Contains user-space application code. Processes are compiled into self-contained ELF executables that must statically link the shared library (`shared/libshared.a`).

**Compilation Process:**
1. User code is compiled into `.elf` and `.bin` formats
2. Binaries are automatically moved to `fs/osbyte/user/`
3. The `createfs` script generates a filesystem image from the `fs/` directory

**Current Limitations:**
- Processes must be entirely self-contained (no dynamic linking)
- Maximum of 9 processes due to array and desktop UI limitations (to be addressed in future versions)
- Only ELF format is currently supported

**Future Plans:** This directory may be replaced by standalone projects for individual user processes.

### Support Scripts

Utility scripts in the project root:

| Script | Description |
|--------|-------------|
| `run_virt` | Runs Osbyte OS in QEMU using the virt board |
| `run_raspi` | Runs Osbyte OS in QEMU emulating Raspberry Pi 4B |
| `debug` | Attaches GDB to a running QEMU instance (requires `./run debug`) |
| `rundebug` | Shortcut to run OS and attach GDB simultaneously |
| `createfs` | Creates filesystem image from `fs/` directory (macOS/Linux) |

**Makefile Targets:**

| Target | Description |
|--------|-------------|
| `make all` | Compiles the system and creates filesystem image |
| `make run` | Compiles and runs the system (default: virt board) |
| `make run MODE=virt` | Compiles and runs for QEMU virt board |
| `make run MODE=raspi` | Compiles and runs for Raspberry Pi 4B |
| `make debug` | Compiles, creates filesystem, and launches with GDB |
| `make clean` | Removes compiled files |
| `make install` | Performs clean build and installs to Raspberry Pi SD card |

## Prerequisites

### Required Tools

1. **ARM GNU Toolchain**: ARM provides pre-built binaries for various host systems
   - Download from: [ARM Developer](https://developer.arm.com/tools-and-software/open-source-software/developer-tools/gnu-toolchain)

2. **QEMU**: Required for emulation
   - Installation varies by platform:
     - macOS: `brew install qemu`
     - Linux: `sudo apt-get install qemu-system-arm` (Debian/Ubuntu)
     - Windows: Download from [QEMU website](https://www.qemu.org/download/)

3. **Make**: Build system (usually pre-installed on Unix-like systems)

4. **GDB**: For debugging (usually included with ARM toolchain)

### Filesystem Setup

Create a `fs/` directory in the project root with the following structure:
```
fs/
└── osbyte/
    └── user/
        └── *.elf  (user process executables)
```

The build process automatically places compiled user processes in this directory.

## Building and Running

### Quick Start

1. **Clone the repository:**
   ```bash
   git clone <repository-url>
   cd RedactedOS-main
   ```

2. **Build and run (QEMU virt board):**
   ```bash
   make run
   ```

3. **Build and run (Raspberry Pi 4B emulation):**
   ```bash
   make run MODE=raspi
   ```

4. **Build only:**
   ```bash
   make all
   ```

### Debugging

To debug the operating system:

1. **Using rundebug script:**
   ```bash
   make debug
   # Or with a breakpoint:
   ./rundebug b kernel_main
   ```

2. **Manual debugging:**
   ```bash
   ./run_virt debug  # In one terminal
   ./debug b kernel_main  # In another terminal
   ```

### Installing on Raspberry Pi

1. Format an SD card using Raspberry Pi Imager, creating a `bootfs` partition
2. Run the install command:
   ```bash
   make install
   ```
3. Insert the SD card into your Raspberry Pi and boot

**Note:** For debugging on physical hardware, use a UART debug probe and connect via `screen`:
```bash
screen /dev/ttyUSB0 115200
```

### Continuous Integration

GitHub Actions automatically compiles the `main` branch on macOS. Pre-built artifacts are available in the Actions tab, including the `fs/` directory. Note that you must run `createfs` locally to regenerate `disk.img` if you modify the filesystem contents.

## Platform Support

### QEMU virt Board (Recommended)

- **Status**: Fully supported
- **Performance**: Excellent
- **Features**: All features available
- **Networking**: Full support
- **Recommended for**: Development and testing

### QEMU Raspberry Pi 4B Emulation

- **Status**: Supported with limitations
- **Performance**: Slower than virt board
- **Limitations**:
  - No networking capabilities
  - Polling-based keyboard input (no interrupts)
  - No DMA for disk transfers (may freeze during data loading)
- **Use case**: Testing Raspberry Pi-specific code paths

### Physical Raspberry Pi Hardware

- **Status**: Preliminary support
- **Tested Platforms**: Raspberry Pi 4B (QEMU only), Raspberry Pi 5 (theoretical)
- **Untested**: Raspberry Pi 3B (theoretical support)
- **Not Supported**: Raspberry Pi 2 and earlier (32-bit only)
- **Recommendation**: Use UART debugging for development

## Networking

Osbyte implements a comprehensive networking stack following a layered architecture.

### Network Interface Management

At startup, the system:
- Scans for available network interface cards (NICs)
- Loads appropriate drivers
- Creates logical L2 interfaces for each device
- Initializes loopback interfaces for both IPv4 and IPv6

### Link Layer

- **Neighbor Resolution**: ARP for IPv4, NDP (Neighbor Discovery Protocol) for IPv6
- **Multicast Filtering**: Traffic filtered at NIC level to reduce unnecessary processing
- **Group Management**: IGMP for IPv4, MLD (Multicast Listener Discovery) for IPv6

### Network Layer

- **IPv4**: Full protocol support with routing
- **IPv6**: Solid implementation covering core features
  - Routing support
  - ICMP/ICMPv6 control protocols
  - **Missing features**: Full SLAAC support, router preference handling, complete extension header support, ULA (Unique Local Addresses)

### Address Configuration

- **IPv4**: DHCP client
- **IPv6**: DHCPv6 (stateful and stateless) and SLAAC support
- Automatic interface configuration without application-level handling

### Transport Layer

- **UDP**: Full support
- **TCP**: Full support with socket-oriented interface

### Application Layer

- **DNS**: Resolver with internal caching to reduce latency and network interrupts
- **Service Discovery**: DNS-SD/mDNS responders and SSDP utilities (currently disabled)
- **NTP**: Network Time Protocol implementation
  - Focuses on accurate offset calculation and stable clock adjustments
  - Not fully standards-compliant
  - SNTP support available but deprecated
- **HTTP**: Client and server implementations
  - Embedded HTTP server listens on port 80
  - Network discovery via UDP broadcast on port 8080

### Example: Firmware Server

An implementation of the HTTP server can be found at the [OsbyteOS Firmware Server Repository](https://github.com/differrari/OsbyteOS_firmware_server/tree/main).

## Development

### Code Style

The codebase uses a mix of C and C++. Kernel code is primarily C, while some device drivers and UI components use C++.

### Adding User Processes

1. Create your process source code
2. Compile it as a self-contained ELF executable
3. Statically link `shared/libshared.a`
4. Place the `.elf` file in `fs/osbyte/user/`
5. Run `createfs` to regenerate the filesystem image
6. Rebuild and run the system

### Debugging Tips

- Use `make debug` for integrated debugging
- Set breakpoints in GDB: `b function_name`
- Use `info registers` to inspect CPU state
- Check kernel logs via serial output (QEMU) or UART (physical hardware)

### Known Issues and Future Improvements

- Kernel processes architecture needs modularization
- Dynamic linking support for user processes
- Expand UI library with more components
- Improve Raspberry Pi hardware support
- Remove fixed-size limitations in data structures
- Full IPv6 feature compliance

## License

See `COPYING` and `GPL` files for license information.
