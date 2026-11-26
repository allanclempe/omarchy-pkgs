# ARM static QEMU

This aarch64-only split recipe provides qemu-user-static and qemu-user-static-binfmt, built from the upstream QEMU release with Arch's static configure options. It preserves Marcelo’s recipe, imported through omarchy-mac/omarchy-pkgs-aarch64#61, including licensing and binfmt behavior. It does not change x86 packages or the build workers.

## Patches

- `1000` keeps 64-bit guest mappings inside the guest's address space. Without it the arm64 kernel places x86-64 mappings near 2^48, which x86-64 code takes for kernel addresses. QEMU now chooses the address, as it does with a reserved address space: it searches the guest page table for a hole below 2^47, claims it with `MAP_FIXED_NOREPLACE`, skips past QEMU's own host mappings, and reuses freed space. `mremap` resizes in place when it can. Mappings start at 3/4 of the guest's 47 bits, as the default x86-64 layout places them.
- `1001` passes Linux AIO (`io_setup`, `io_submit`, `io_getevents`, `io_cancel`, `io_destroy`) through to the host when guest and host share the structure layout; stock qemu-user returns ENOSYS. Every iocb, buffer and event array is checked against guest memory first, a read into a page QEMU protected for translated code is redone, and result errnos are translated.
- `1002` is upstream QEMU `586284af4c2c`, carried unchanged: `sigcontext.trapno` carries the real trap number instead of -1. QEMU 11.2 has it; drop the patch then.

## Updates

Updates are reviewed pins, not automatic. To move to a new QEMU release, update `pkgver` and the tarball's `sha256sums` entry together, check its detached signature against Michael Roth's key `CEACC9E15534EBABB82D3FA03353C9CEF108B584` outside the build, and rebase the patches. The epoch of 1 stays from the earlier Debian repackaging, so upstream versions sort above it. The build fails if a binfmt rule names an emulator the package doesn't ship.

## binfmt rules

The rules are generated with `--ignore-family yes`, so 32-bit ARM binaries are registered even though the script groups them with aarch64; Apple Silicon has no AArch32 execution. Native aarch64 stays excluded. Rules use the persistent and preserve-argv0 flags and never the credential flag.
