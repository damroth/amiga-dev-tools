# Maintainer: Damroth
pkgname=amiga-dev-tools
pkgver=1.0
pkgrel=1
pkgdesc="Cross-compilation toolchain for Amiga (vasm, vbcc, vlink, NDK 3.2)"
arch=('x86_64')
url="https://github.com/damroth/amiga-dev-tools"
license=('MIT')
depends=('glibc' 'lha')
makedepends=('wget' 'unzip' 'tar')
install="${pkgname}.install"
source=(
    "http://server.owl.de/~frank/tags/vasm2_0d.tar.gz"
    "http://server.owl.de/~frank/tags/vbcc0_9hP3.tar.gz"
    "http://server.owl.de/~frank/tags/vlink0_18a.tar.gz"
    "NDK3.2.lha::https://aminet.net/dev/misc/NDK3.2.lha"
    "vbcc_target_m68k-amigaos.lha::http://phoenix.owl.de/vbcc/current/vbcc_target_m68k-amigaos.lha"
    "http://phoenix.owl.de/vbcc/current/vbcc_unix_config.tar.gz"
)
sha256sums=('SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP')

_prefix="/opt/amiga-dev"

# Don't auto-extract vbcc due to hardlink issues
noextract=('vbcc0_9hP3.tar.gz')

prepare() {
    cd "$srcdir"

    # Manually extract vbcc with hardlink fix
    if [ ! -d "vbcc" ]; then
        tar -xzf "${SRCDEST:-$startdir}/vbcc0_9hP3.tar.gz" --hard-dereference 2>/dev/null || \
        bsdtar -xf "${SRCDEST:-$startdir}/vbcc0_9hP3.tar.gz" 2>/dev/null || true
    fi
}

build() {
    cd "$srcdir/vasm"
    make CPU=m68k SYNTAX=mot

    # Build vbcc - provide many 'y' answers for dtgen questions
    # dtgen asks multiple questions about host system types, all default to 'y'
    cd "$srcdir/vbcc"
    mkdir -p bin
    yes "" | timeout 60 make TARGET=m68k || make TARGET=m68k

    cd "$srcdir/vlink"
    make
}

package() {
    # Create installation directories
    install -dm755 "$pkgdir/$_prefix"/{bin,targets/m68k-amigaos,ndk}

    # Install vasm
    cd "$srcdir/vasm"
    install -Dm755 vasmm68k_mot "$pkgdir/$_prefix/bin/vasmm68k_mot"
    install -Dm755 vobjdump "$pkgdir/$_prefix/bin/vobjdump" 2>/dev/null || true

    # Install vbcc
    cd "$srcdir/vbcc"
    install -Dm755 bin/vbccm68k "$pkgdir/$_prefix/bin/vbccm68k"
    install -Dm755 bin/vc "$pkgdir/$_prefix/bin/vc"
    install -Dm755 bin/vprof "$pkgdir/$_prefix/bin/vprof" 2>/dev/null || true

    # Install vlink
    cd "$srcdir/vlink"
    install -Dm755 vlink "$pkgdir/$_prefix/bin/vlink"

    # Install vbcc config files for Unix
    cd "$srcdir"
    if [ -d "config" ]; then
        install -dm755 "$pkgdir/$_prefix/etc"
        cp -r config/* "$pkgdir/$_prefix/etc/" 2>/dev/null || true
    fi

    # Install vbcc target files
    cd "$srcdir"
    if [ -d "vbcc_target_m68k-amigaos" ]; then
        cp -r vbcc_target_m68k-amigaos/* "$pkgdir/$_prefix/targets/m68k-amigaos/"
    fi

    # Install NDK
    # Try different possible directory names from NDK extraction
    for ndk_dir in "NDK_3.2" "NDK3.2" "ndk3.2" "Include" "." ; do
        if [ -d "$srcdir/$ndk_dir/Include" ]; then
            cp -r "$srcdir/$ndk_dir"/* "$pkgdir/$_prefix/ndk/" 2>/dev/null || true
            break
        fi
    done

    # If Include directory is directly in srcdir, copy it
    if [ -d "$srcdir/Include" ]; then
        cp -r "$srcdir/Include" "$pkgdir/$_prefix/ndk/" 2>/dev/null || true
    fi
    if [ -d "$srcdir/lib" ]; then
        cp -r "$srcdir/lib" "$pkgdir/$_prefix/ndk/" 2>/dev/null || true
    fi

    # Create environment setup script for bash/zsh
    install -dm755 "$pkgdir/etc/profile.d"
    cat > "$pkgdir/etc/profile.d/amiga-dev.sh" << 'EOF'
# Amiga development tools environment
export AMIGA_DEV="/opt/amiga-dev"
export PATH="$AMIGA_DEV/bin:$PATH"
export VBCC="$AMIGA_DEV"
export VBCCCONFIG="$AMIGA_DEV/etc/vc.config"
export NDK="$AMIGA_DEV/ndk"
EOF

    # Create environment setup script for fish
    install -dm755 "$pkgdir/usr/share/fish/vendor_conf.d"
    cat > "$pkgdir/usr/share/fish/vendor_conf.d/amiga-dev.fish" << 'EOF'
# Amiga development tools environment
set -gx AMIGA_DEV "/opt/amiga-dev"
set -gx PATH "$AMIGA_DEV/bin" $PATH
set -gx VBCC "$AMIGA_DEV"
set -gx VBCCCONFIG "$AMIGA_DEV/etc/vc.config"
set -gx NDK "$AMIGA_DEV/ndk"
EOF

    # Install documentation
    install -dm755 "$pkgdir/usr/share/doc/$pkgname"
    echo "Amiga Cross-Compilation Toolchain" > "$pkgdir/usr/share/doc/$pkgname/README"
    echo "" >> "$pkgdir/usr/share/doc/$pkgname/README"
    echo "Environment variables are set automatically via /etc/profile.d/amiga-dev.sh" >> "$pkgdir/usr/share/doc/$pkgname/README"
    echo "" >> "$pkgdir/usr/share/doc/$pkgname/README"
    echo "Tools installed:" >> "$pkgdir/usr/share/doc/$pkgname/README"
    echo "  - vasmm68k_mot: Motorola 68k assembler" >> "$pkgdir/usr/share/doc/$pkgname/README"
    echo "  - vbccm68k: VBCC compiler for m68k" >> "$pkgdir/usr/share/doc/$pkgname/README"
    echo "  - vc: VBCC frontend" >> "$pkgdir/usr/share/doc/$pkgname/README"
    echo "  - vlink: Linker" >> "$pkgdir/usr/share/doc/$pkgname/README"
    echo "" >> "$pkgdir/usr/share/doc/$pkgname/README"
    echo "NDK 3.2 installed in: \$AMIGA_DEV/ndk" >> "$pkgdir/usr/share/doc/$pkgname/README"
}
