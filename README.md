# Amiga Development Tools

Cross-compilation toolchain for Amiga m68k development on Linux.

## Contents

- **vasm** - Motorola 68000 assembler
- **vbcc** - ISO C compiler for m68k
- **vlink** - Object linker
- **NDK 3.2** - Native Development Kit

## Installation

```bash
paru -S git+https://github.com/damroth/amiga-dev-tools.git
```

Or build locally:

```bash
git clone https://github.com/damroth/amiga-dev-tools.git
cd amiga-dev-tools
makepkg -si
```

After installation, reload your shell:

```bash
exec $SHELL
```

## Usage

Assemble:
```bash
vasmm68k_mot -Fhunk -o hello hello.s
```

Compile:
```bash
vc +aos68k -o hello hello.c
```

## Environment

The package sets these automatically:

- `AMIGA_DEV=/opt/amiga-dev`
- `VBCC=/opt/amiga-dev`
- `NDK=/opt/amiga-dev/ndk`

Tools are in `/opt/amiga-dev/bin`.

