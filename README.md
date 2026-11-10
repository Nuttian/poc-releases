# Nuttian proof-of-concept releases

Nuttian is a distribution built on [Apache NuttX](https://nuttx.apache.org/).
Its programs are ordinary binary packages that install with dpkg and apt,
on a NuttX kernel build: each program is a process in its own address
space, linked against NuttX's own C library as `libc.so`.

This repository holds the **proof-of-concept releases**: ready-to-boot
images for QEMU. The packages themselves come from
`https://archive.nuttian.org/`. More: <https://nuttian.org/>.

## Releases

The current release is **poc-15.1**. Each binary interface (ABI) of the
kernel and the C library has its own archive tree on archive.nuttian.org, so a
system never mixes two; poc-15.1 uses `poc-15`. A newer release of the same ABI
(a point release) replaces the packages of its tree. We keep only the current
release: when a new ABI comes out, the older images and their archive tree are
deleted, so an older image can no longer install packages: download the new
image. Each release has one download per architecture:

| Architecture | Download | Runs fast on |
|---|---|---|
| 64-bit Arm | `nuttian-nxsid-arm64.tar.xz` | Apple silicon Macs; arm64 Linux (KVM) |
| x86-64 | `nuttian-nxsid-amd64.tar.xz` | Intel Macs; x86-64 Linux (KVM); Windows with WSL 2 |
| RISC-V 64 | `nuttian-nxsid-riscv64.tar.xz` | emulated everywhere for now |

Every architecture runs everywhere; where it does not match your computer,
QEMU emulates it (slower). `SHA256SUMS` lists the checksums of the
downloads, and `SHA256SUMS.asc` is its signature with the Nuttian archive
key (the same key that signs the apt archive).

Tested on Apple silicon Macs and on x86-64 Linux. Intel Macs and WSL 2 should
work the same way, but we have not tested them yet: please tell us.

## Try it

1. Install QEMU (on Linux you also need `python3`):

   ```sh
   brew install qemu                                    # macOS
   sudo apt install qemu-system-arm qemu-system-x86 qemu-system-misc   # Debian, Ubuntu
   sudo dnf install qemu-system-aarch64 qemu-system-x86 qemu-system-riscv  # Fedora
   ```

   Windows: use WSL 2 (`wsl --install -d Ubuntu` in PowerShell), then the
   Debian, Ubuntu line in Ubuntu. QEMU for Windows has no shared folders
   (9p), which the root file system uses.

2. Download and start:

   ```sh
   ARCH=arm64     # or amd64, riscv64
   curl -LO https://archive.nuttian.org/releases/poc-15.1/nuttian-nxsid-$ARCH.tar.xz
   tar xf nuttian-nxsid-$ARCH.tar.xz
   cd nuttian-nxsid-$ARCH
   ./run.sh
   ```

   The same files are attached to the [latest release](https://github.com/Nuttian/poc-releases/releases/latest)
   here.

   `run.sh` finds QEMU, uses hardware acceleration when it can, and on
   macOS puts the root file system on a case-sensitive disk image. Run it
   again to come back to the same system.

3. In the guest (a `bash` login shell as root):

   ```sh
   apt update
   apt install build-essential
   ```

   To leave, run `poweroff`, or press `Ctrl-a`, then `x`.

   For the desktop, start with `./run.sh --gui`: it comes up by itself, with a
   panel of icons: a terminal, the `lite-xl` editor and Freedoom. `./run.sh --help`
   lists the other options, such as `--mem 4G` and `--res 1920x1080`.

## Know before you try

- This is a proof of concept. Do not use it for anything that matters.
- The binary interface can change between proof-of-concept releases.
- The root file system is a folder on your computer, shared into the guest.
  Start one guest per folder.
- A small desktop with sound: `./run.sh --gui`. OpenGL and OpenGL ES in software
  (Mesa); `./run.sh --gui --gl` draws on the host's GPU where QEMU has virgl
  (most Linux distributions; not Homebrew's QEMU on macOS). GTK 3, Qt 6, Tk and
  X11 programs install with apt. No graphical web browser. Sound needs CoreAudio on macOS, or a PipeWire or PulseAudio server
  on Linux; otherwise the desktop runs silently.
- Some Linux-only tools (procps, iproute2) have small NuttX replacements
  (`nuttian-tools`).
- `./share`, next to `run.sh`, is a folder of your computer that the guest can
  use too: `mount -t v9fs -o trans=virtio,tag=host /mnt` in the guest.

## Source code

Every binary package in the images and in the archive comes from a source
package in the same archive: <https://archive.nuttian.org/poc-15/index.html>
lists them all, with their files. Each package's sheet on
<https://packages.nuttian.org/> links its source files and lists what Nuttian
changes for NuttX, and why.

The images have `deb-src` in their source list. To get the source in the
guest, work in a directory on the root file system (`/tmp` is small):

```sh
apt install dpkg-dev
mkdir -p ~/src && cd ~/src
apt source <package>
```

## Reports and contact

- Questions, reports, ideas: the issues of this repository, or
  dev@nuttian.org.
- Security issues: security@nuttian.org (see [SECURITY.md](SECURITY.md)).

## Licence

The files of this repository are under the Apache License 2.0
([LICENSE](LICENSE)). The images contain many programs under their own
licences; each package lists them in `/usr/share/doc/<package>/copyright`.

Nuttian is not a project of the Apache Software Foundation. Apache, Apache
NuttX and NuttX are trademarks of The Apache Software Foundation.
