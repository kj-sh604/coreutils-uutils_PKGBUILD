_pkgname=coreutils
__pkgname=uutils-$_pkgname
pkgname=rust-uutils-as-$_pkgname
pkgver=0.0.24
pkgrel=1
pkgdesc='Cross-platform Rust rewrite of the GNU coreutils being used as actual coreutils'
arch=('x86_64')
url='https://github.com/uutils/coreutils'
license=('MIT')
depends=('glibc' 'gcc-libs')
makedepends=('rust' 'cargo' 'python-sphinx')
source=("$__pkgname-$pkgver.tar.gz::$url/archive/$pkgver.tar.gz")
sha512sums=('da9028effede4e925263244f0fdcfdd13f4d44a4baf2da57df090aad8c3821b880a10dbb74d8e1e2958f324299f63ebdbd1bb068895c000835b1bb12fcccc599')
options=('!lto')
provides=('coreutils')
conflicts=('coreutils' 'coreutils-hybrid')

prepare() {
  cd $_pkgname-$pkgver
  sed 's|"bin"|"builduser"|g' -i tests/by-util/test_{chgrp,chown}.rs
}

build() {
  cd $_pkgname-$pkgver

  make PROFILE=release
}

# https://github.com/uutils/coreutils/issues/4946
# check() {
#   cd $_pkgname-$pkgver
#
#   make test \
#       PROFILE=release \
#       CARGOFLAGS=--release \
#       TEST_NO_FAIL_FAST="--no-fail-fast -- \
#         --skip test_chown::test_big_p \
#         --skip test_chgrp::test_big_p \
#         --skip test_chgrp::test_big_h"
# }

package() {
  cd $_pkgname-$pkgver

  make \
      DESTDIR="$pkgdir" \
      PREFIX=/usr \
      MANDIR=/share/man/man1 \
      PROG_PREFIX= \
      install

  install -Dm 644 LICENSE "$pkgdir"/usr/share/licenses/$pkgname/LICENSE

  # clean conflicts, arch ships these in other apps
  cd $pkgdir/usr/bin
  rm groups hostname kill more uptime
  rm $pkgdir/usr/share/bash-completion/completions/*
  rm $pkgdir/usr/share/man/man1/{groups.1,hostname.1,kill.1,more.1,uptime.1}
}

# vim: ts=2 sw=2 et:
