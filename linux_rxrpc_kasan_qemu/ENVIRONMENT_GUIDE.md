# Linux RxRPC KASAN: agent environment guide

This guide starts after the platform has given the agent a shell in a ready Linux runtime container. The container is the QEMU host; the agent starts the kernel guest when a test needs it. The built kernel corresponds to source commit `7167dde192f2c5807fa2163bf16013a12b0d3764`.

## Working locations

| Path | Purpose |
| --- | --- |
| `/runtime-base/build/arch/x86/boot/bzImage` | Prebuilt KASAN kernel for QEMU. |
| `/runtime-base/build/vmlinux` | Unstripped kernel for symbols and debugging. |
| `/runtime-base/rootfs.cpio` | BusyBox initramfs; the guest sets up loopback and includes RxGK lab fixtures under `/opt/brainafk/rxgk-lab`. |
| `/runtime-base/start-qemu.sh` | Starts QEMU with the bundled kernel and an initramfs. |
| `/runtime-base/scripts` | PoC initramfs, command, and smoke-test helpers. |
| `/runtime-base/artifacts/kernel.config` | Config used for the prebuilt kernel. |
| `/artifacts` | Writable location for a PoC initramfs and serial logs. |
| `/workspace/code` | Writable task workspace when a matching Linux source checkout is mounted. |
| `/opt/brainafk/preset` | Original preset resources and builder. |

## Dynamic testing

1. Check the baseline with `/runtime-base/scripts/smoke-test.sh`. It boots QEMU and verifies the guest and loopback startup markers.
2. For a guest PoC, prepare an executable guest init script, for example `/artifacts/poc-init.sh`. Add any required guest files through the helper's optional overlay directory.
3. Build and boot a PoC initramfs:

   ```sh
   /runtime-base/scripts/build-poc-initramfs.sh \
     /artifacts/poc-init.sh /artifacts/poc-rootfs.cpio
   /runtime-base/start-qemu.sh \
     --initramfs /artifacts/poc-rootfs.cpio \
     --serial-log /artifacts/poc-serial.log
   ```

4. Inspect the serial log for the PoC exit status, kernel diagnostics, and any KASAN report. Preserve the PoC input and the log as evidence; a successful QEMU boot alone does not prove the target path was exercised.

QEMU uses `/dev/kvm` when available and otherwise runs in software emulation. The image contains the compiled kernel and runtime helpers, but not the full Linux source tree. Kernel changes and rebuilds require a matching writable source checkout mounted at `/workspace/code`. RxGK fixtures are lab inputs, not a full Kerberos service or a validated encrypted handshake.
