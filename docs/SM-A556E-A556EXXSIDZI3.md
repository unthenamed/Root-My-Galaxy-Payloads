# SM-A556E A556EXXSIDZI3 target record

## Firmware identity

```text
model: SM-A556E
device: a55x
display build: BP4A.251205.006.A556EXXSIDZI3
fingerprint: samsung/a55xnsxx/a55x:16/BP4A.251205.006/A556EXXSIDZI3:user/release-keys
kernel release: 6.1.157-android14-11
kernel version: Linux version 6.1.157-android14-11 (kleaf@build-host) (Android (11368308, +pgo, +bolt, +lto, +mlgo, based on r510928) clang version 18.0.0 (https://android.googlesource.com/toolchain/llvm-project 477610d4d0d988e69dbc3fae4fe86bff3f07f2b5), LLD 18.0.0) #1 SMP PREEMPT Mon Sep  7 11:02:17 UTC 2026
SDK: 34
ABI: arm64-v8a
page size: 4096
```

## Target derivation

The local `boot.img` was unpacked and recovered to:

```text
ida/boot-kernel.bin
ida/vmlinux.elf
ida/vmlinux.btf
ida/vmlinux.nm
```

IDA was kept at imagebase 0. Runtime kernel addresses are therefore
`KIMAGE_TEXT_BASE + kaslr_slide + *_OFF`.

Important target constants:

```text
KIMAGE_TEXT_BASE: 0xffffffc008000000
P0_PAGE_OFFSET:   0xffffff8000000000
P0_PHYS_OFFSET:   0x80000000
DIRECT_MAP_BASE:  0xffffff8000000000
VMEMMAP_START:    0xfffffffe00000000
sched_blocked_reason tracepoint id: 106
worker_thread post-schedule caller offset: 0x000dbc94
```

The P0 oracle fingerprint table maps the actual slide to bytes read at raw
offset `P0_ORACLE_PROBE_OFFSET - slide`. Without this conversion the raw sample
is misread as the slide and the fops direct-map verification reads zero.

## Exploit validation

The A55 app-domain payload was built as:

```text
artifacts/a55x-A556EXXSIDZI3/cve-2026-43499-app.release.so
size: 104128
SHA-256: (pending hardware test)
```

Hardware execution pending on SM-A556E. The exploit uses the physical P0 oracle
route with tracefs-assisted slide discovery and KernelSnitch for deterministic
SKB reclaim.

## KernelSU build and validation record

KernelSU was built from tag `v3.2.5`, commit
`b0bc817b4e966aa6aa830834eaf6ef765d821d40`, with the repository Samsung
KDP/RKP/DEFEX patch.

The module was built in:

```text
ghcr.io/ylarod/ddk-min:android14-6.1-20260313
```

The DDK release was overridden to the exact target release:

```text
vermagic: 6.1.157-android14-11 SMP preempt mod_unload modversions aarch64
```

The target config has:

```text
CONFIG_MODULES=y
CONFIG_MODVERSIONS=y
CONFIG_MODULE_SIG_ALL=y
# CONFIG_MODULE_SIG_FORCE is not set
# CONFIG_MODULE_FORCE_LOAD is not set
CONFIG_TRIM_UNUSED_KSYMS=y
CONFIG_KALLSYMS_ALL=y
CONFIG_KDP=y
CONFIG_RKP=y
CONFIG_SECURITY_DEFEX=y
```

For this Exynos target, the A55 module is built with:

```text
CONFIG_KSU_SAMSUNG_NO_PATCH_TEXT=y
```

That target flag makes KernelSU refuse `ksu_patch_text()` live writes instead
of entering `stop_machine`. The syscall-table dispatcher then falls through to
the Samsung RKP kretprobe/kprobe fallback, and selinux-hide slot patches fail
closed instead of triggering Samsung EL2.

The stripped standalone KO is:

```text
kernelsu/android14-6.1_kernelsu-A556EXXSIDZI3-kdp.ko
size: 400112
SHA-256: (pending)
```

Static checks passed:

```text
__versions size: 0
undefined symbols: (pending)
module version entries: 0
missing from target symbol table: 0
symbols resolved from kallsyms rather than target exports: (pending)
target CRC mismatches: 0
```

The Android/AArch64 `ksud` binary embeds the A55 KO as
`android14-6.1_kernelsu.ko`:

```text
kernelsu/ksud-A556EXXSIDZI3-kdp
size: 6532168
SHA-256: (pending)
```

The current artifact has been built and pushed to the fork. Hardware validation
on SM-A556E is pending. Once tested, this document will be updated with
SHA-256 hashes and execution results.

This profile is exact-build support; it does not claim compatibility with
other Galaxy A55 models, firmware, or kernel releases.