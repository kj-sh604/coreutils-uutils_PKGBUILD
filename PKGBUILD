pkgname=coreutils-uutils
pkgver=0.0.24
pkgrel=604
gnu_coreutils=coreutils
rust_uutils=uutils-coreutils
gnu_coreutils_version=9.4
rust_uutils_version=0.0.24
pkgdesc='Cross-platform Rust rewrite of the GNU coreutils being used as actual system coreutils'
arch=('x86_64')
license=('GPL3' 'MIT')
url='https://www.gnu.org/software/coreutils/'
_url='https://github.com/uutils/coreutils'
depends=('glibc' 'acl' 'attr' 'gmp' 'libcap' 'openssl' 'gcc-libs' 'libkeccak')
conflicts=('coreutils' 'coreutils-hybrid' 'coreutils-hybrid-git' 'b3sum' 'sha3sum')
provides=('coreutils' 'b3sum' 'sha3sum')
makedepends=('rust' 'cargo' 'python-sphinx')
source=("https://ftp.gnu.org/gnu/$gnu_coreutils/$gnu_coreutils-$gnu_coreutils_version.tar.xz"
        "$rust_uutils-$rust_uutils_version.tar.gz::$_url/archive/$rust_uutils_version.tar.gz")
sha512sums=('7c55ee23b685a0462bbbd118b04d25278c902604a0dcf3bf4f8bf81faa0500dee5a7813cba6f586d676c98e520cafd420f16479619305e94ea6798d8437561f5'
            'da9028effede4e925263244f0fdcfdd13f4d44a4baf2da57df090aad8c3821b880a10dbb74d8e1e2958f324299f63ebdbd1bb068895c000835b1bb12fcccc599')

prepare() {
  cd $gnu_coreutils-$gnu_coreutils_version
  # apply patch from the source array (should be a pacman feature)
  local filename
  for filename in "${source[@]}"; do
    if [[ "$filename" =~ \.patch$ ]]; then
      echo "Applying patch ${filename##*/}"
      patch -p1 -N -i "$srcdir/${filename##*/}"
    fi
  done
  :
}

build() {
  # build gnu coreutils that are not implemented in the MULTICALL rust uutils 
  cd $gnu_coreutils-$gnu_coreutils_version
  ./configure \
      --prefix=/usr \
      --libexecdir=/usr/lib \
      --with-openssl \
      --enable-no-install-program="[,arch,b2sum,base32,base64,basename,basenc,cat,\
      chgrp,chmod,chown,chroot,cksum,comm,cp,csplit,cut,date,dd,df,dir,dircolors,\
      dirname,du,echo,env,expand,expr,factor,false,fmt,fold,head,hostid,id,\
      join,link,ln,logname,ls,md5sum,mkdir,mkfifo,mknod,mktemp,mv,nice,nl,nohup,\
      nproc,numfmt,od,paste,pathchk,pinky,pr,printenv,printf,ptx,pwd,readlink,\
      realpath,rm,rmdir,seq,sha1sum,sha224sum,sha256sum,sha384sum,sha512sum,\
      shred,shuf,sleep,sort,split,stat,stdbuf,sum,sync,tac,tail,tee,test,timeout,\
      touch,tr,true,truncate,tsort,tty,uname,unexpand,uniq,unlink,users,vdir,wc,\
      who,whoami,yes"
}

package() {
  cd $gnu_coreutils-$rust_uutils_version
  make \
      DESTDIR="$pkgdir" \
      PREFIX=/usr \
      MANDIR=/share/man/man1 \
      PROG_PREFIX= \
      PROFILE=release \
      MULTICALL=y \
      install
  # merge specified gnu coreutils with the rust uutils 
  cd $srcdir && cd $gnu_coreutils-$gnu_coreutils_version
  make DESTDIR="$pkgdir" install
  # add libstdbuf.so
  mkdir -p $pkgdir/usr/lib/coreutils
  cd $srcdir && cd $gnu_coreutils-$rust_uutils_version/target/release/deps
  mv liblibstdbuf.so $pkgdir/usr/lib/coreutils/libstdbuf.so
  # clean conflicts, arch ships these in other apps
  cd $pkgdir/usr/bin
  rm groups hostname kill more uptime
  # symlink missing binaries
  if [ -f "coreutils" ]; then
    local binaries=(
      "b2sum" "b3sum" "md5sum" "sha1sum" "sha224sum"
      "sha256sum" "sha3-224sum" "sha3-256sum" "sha3-384sum" "sha3-512sum"
      "sha384sum" "sha3sum" "sha512sum" "shake128sum" "shake256sum"
    )
    for bin in "${binaries[@]}"; do
      ln -s coreutils "$bin" || echo "warning: failed to create symlink for $bin"
    done
  else
    echo "coreutils binary not found, skipping symlink creation."
  fi
  # additional cleanup
  rm $pkgdir/usr/share/bash-completion/completions/*
  rm $pkgdir/usr/share/man/man1/{groups.1,hostname.1,kill.1,more.1,uptime.1}
}
