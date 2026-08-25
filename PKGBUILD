# Maintainer: Kestrel Arch Core Team
# Description: Kestrel Arch High-Performance Kernel (Clang ThinLTO, Native Rust, BORE)

pkgbase=linux-kestrel
_major=6.13
_minor=4
pkgver=${_major}.${_minor}
pkgrel=1
pkgdesc='Linux Kestrel High-Performance Kernel with Clang ThinLTO and Rust Support'
arch=('x86_64')
url="https://github.com/Kestrel-Arch/linux-kestrel"
license=('GPL2')
options=('!strip' '!debug')

# =================================================================
# KESTREL ARCHITECTURE MATRIX
# Options: "generic" (v1), "v2", "v3", "v4", "zen4", "zen5"
# Defaulting to "generic" ensures boot compatibility on ALL hardware.
# =================================================================
_kestrel_arch="generic"

if [ "$_kestrel_arch" != "generic" ]; then
    pkgbase="linux-kestrel-${_kestrel_arch}"
fi

makedepends=(
  'bc' 'cpio' 'gettext' 'libelf' 'pahole' 'perl' 'python' 'tar' 'xz' 'kmod'
  'clang' 'llvm' 'lld' 'rust' 'rust-src' 'rust-bindgen' 'git'
)

source=(
  "https://cdn.kernel.org/pub/linux/kernel/v6.x/linux-${pkgver}.tar.xz"
  "config"
  "linux-kestrel.preset"
)

# Skipping checksums to allow dynamic editing of config and presets
sha256sums=('SKIP' 'SKIP' 'SKIP')

prepare() {
  cd "linux-${pkgver}"

  # 1. Apply any custom patches found in the Patches/ directory
  if [ -d "$srcdir/Patches" ]; then
    echo "==> Scanning for Kestrel custom patches..."
    for p in "$srcdir/Patches"/*.patch; do
      if [ -f "$p" ]; then
        echo "  -> Applying patch: $(basename "$p")"
        patch -p1 -i "$p"
      fi
    done
  fi

  # 2. Inject Kestrel configuration
  echo "==> Copying kernel config..."
  cp "$srcdir/config" .config

  # 3. Dynamic Microarchitecture Switcher
  echo "==> Configuring microarchitecture: ${_kestrel_arch}..."
  sed -i -e '/CONFIG_GENERIC_CPU/d' .config
  sed -i -e '/CONFIG_MZEN/d' .config

  case "$_kestrel_arch" in
    v2)   echo "CONFIG_GENERIC_CPU_V2=y" >> .config ;;
    v3)   echo "CONFIG_GENERIC_CPU_V3=y" >> .config ;;
    v4)   echo "CONFIG_GENERIC_CPU_V4=y" >> .config ;;
    zen4) echo "CONFIG_MZEN4=y" >> .config ;;
    zen5) echo "CONFIG_MZEN5=y" >> .config ;;
    *)    echo "CONFIG_GENERIC_CPU=y" >> .config ;;
  esac

  # 4. Set local kernel branding string
  echo "-kestrel" > localversion

  # 5. Lock in the config with LLVM Toolchain
  make LLVM=1 LLVM_IAS=1 olddefconfig
  
  # 6. Generate exact kernel release string
  make -s kernelrelease > version
}

build() {
  cd "linux-${pkgver}"
  
  echo "==> Compiling Kestrel Kernel via LLVM/Clang and Rustc..."
  # LLVM=1 enforces Clang/ThinLTO. LLVM_IAS=1 enforces the LLVM Integrated Assembler
  make LLVM=1 LLVM_IAS=1 -j$(nproc) all
}

package_linux-kestrel() {
  pkgdesc="The Linux Kestrel kernel and core modules"
  depends=('coreutils' 'kmod' 'initramfs')
  optdepends=('linux-firmware: firmware images needed for some devices')
  provides=('linux')

  cd "linux-${pkgver}"
  local kernver="$(<version)"
  local modulesdir="$pkgdir/usr/lib/modules/$kernver"

  echo "==> Installing kernel modules..."
  make LLVM=1 LLVM_IAS=1 INSTALL_MOD_PATH="$pkgdir/usr" INSTALL_MOD_STRIP=1 \
    DEPMOD=/doesnt/exist modules_install

  # Remove symlinks that point to the build directory (pacman handles this)
  rm -f "$modulesdir"/{source,build}

  echo "==> Installing vmlinuz kernel image..."
  install -Dm644 "$(make -s image_name)" "$pkgdir/boot/vmlinuz-linux-kestrel"

  echo "==> Installing mkinitcpio boot preset..."
  install -Dm644 "$srcdir/linux-kestrel.preset" "$pkgdir/etc/mkinitcpio.d/linux-kestrel.preset"
}

package_linux-kestrel-headers() {
  pkgdesc="Headers and scripts for building modules for the Linux Kestrel kernel"
  depends=('pahole')
  provides=('linux-headers')

  cd "linux-${pkgver}"
  local builddir="$pkgdir/usr/lib/modules/$(<version)/build"

  echo "==> Installing headers and Kbuild scripts..."
  install -Dt "$builddir" -m644 .config Makefile Module.symvers System.map \
    localversion* version vmlinux
  
  install -Dt "$builddir/kernel" -m644 kernel/Makefile
  install -Dt "$builddir/arch/x86" -m644 arch/x86/Makefile
  cp -t "$builddir" -a scripts include
  
  # Crucial: Install Rust bindings so native Rust modules can be built later!
  if [ -d "rust" ]; then
    echo "==> Installing Rust bindings..."
    cp -t "$builddir" -a rust
  fi

  # Install Arch-specific headers
  cp -t "$builddir/arch/x86" -a arch/x86/include

  echo "==> Cleaning up unneeded build artifacts from headers..."
  find "$builddir" \( -name '*.o' -o -name '*.a' -o -name '*.cmd' -o -name '.*.cmd' \) -exec rm -f {} +
  rm -f "$builddir/scripts/extable"
}
