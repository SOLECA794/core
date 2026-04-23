# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Structure

This is a multi-submodule OS kernel development project for a Chinese university OS competition (全国大学生OS比赛). It has three submodules:

- **oskernel2026-kksc03** — Student work directory (the primary working directory)
- **StarryOS** — Main kernel, a Rust-based OS built on ArceOS, supports RISC-V64 and LoongArch64
- **testsuits-for-oskernel** — Official test suites for the competition

## Key Commands

### Running in Docker (Required for Competition)

The competition Docker image has all build tools preconfigured. Always run commands inside the container:

```bash
# Run StarryOS on QEMU (from host, runs inside container)
docker run --rm -v $(pwd):/workspace zhouzhouyi/os-contest:20260104 bash -c "cd /workspace/StarryOS && make run ARCH=riscv64"
```

⚠️ **Important**: Never run `make rootfs` — the competition prohibits building rootfs from source. The `disk.img` already exists and is pre-configured with git, vim, gcc, and rustc.

### Build and Run (Local Dev - NOT for Competition)

在 StarryOS 目录下：

```bash
cd StarryOS

# BUS 默认是 pci，所以用 virtio-blk-pci
make run ARCH=riscv64 BLK=y NET=y BUS=pci

# 如果要手动跑 QEMU（调试用）：
qemu-system-riscv64 \
  -machine virt \
  -kernel StarryOS_riscv64-qemu-virt.bin \
  -m 1G -smp 1 -nographic \
  -device virtio-blk-pci,drive=disk0 \
  -drive id=disk0,if=none,format=raw,file=disk.img \
  -device virtio-net-pci,netdev=net0 \
  -netdev user,id=net0
```

内核启动后会显示 `starry:~#` 作为 shell 提示符。

### Code Quality

```bash
cargo fmt
cargo clippy --all-targets --all-features -- -D warnings
```

## Architecture

### StarryOS Kernel Structure (`StarryOS/kernel/src/`)
- **config/** — Kernel configuration
- **file/** — File system implementation
- **mm/** — Memory management
- **sys/** — System call handlers
- **task/** — Task/process management
- **pseudofs/** — Virtual file system structures
- **entry.rs** — Kernel entry point

### Supported Platforms
- **QEMU RISC-V64** with virtio-net/virtio-block (default)
- **QEMU LoongArch64** with virtio-net/virtio-block
- **RISC-V64 VisionFive2** (physical board)
- **LoongArch64 2K1000** (physical board)

### Key Dependencies
- Rust with 2024 edition (see `rust-toolchain.toml`)
- QEMU for emulation
- musl toolchain for cross-compilation

## Commit Convention

Follow Conventional Commits specification:

```
<type>(<scope>): <subject>

feat(syscall): add epoll system call support
fix(mm): resolve page allocation deadlock
```

Types: feat, fix, docs, style, refactor, perf, test, chore

## Competition Tasks

The testsuits are for a competition involving:
1. **git** functionality (clone, push, pull)
2. **vim** editor
3. **gcc** compiler
4. **rustc** compiler

All must run on the custom OS kernel. The TODO.md in oskernel2026-kksc03/docs has detailed task priorities (Ext4, syscalls, process management, networking, SMP).