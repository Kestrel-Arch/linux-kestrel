# Maintainer: Kestrel Arch Core Team

pkgbase=linux-kestrel
_major=6.13
_minor=4
pkgver=${_major}.${_minor}
pkgrel=1
pkgdesc='Linux Kestrel High-Performance Kernel with BORE, Clang ThinLTO, and Rust Support'
url='https://github.com/Kestrel-Arch/linux-kestrel'
arch=('x86_64')
license=('GPL-2.0-only')
options=('!strip')

# Target instruction set: generic (v1), v2, v3, v4, zen4, zen5
_kestrel_arch="${_kestrel_arch:-generic}"

# Core dependencies for Rust + Clang compilation
makedepends=(
  bc
  cpio
  clang
  llvm
  lld
  rust
  rust-src
  rust-bindgen
  kmod
  libelf
  pahole
  python
  tar
  xz
)
