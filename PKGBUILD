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
depends=('glibc' 'acl' 'attr' 'gmp' 'libcap' 'openssl')
conflicts=('coreutils' 'coreutils-hybrid' 'rust-uutils-as-coreutils')
provides=('coreutils')
makedepends=('rust' 'cargo')
source=("https://ftp.gnu.org/gnu/$gnu_coreutils/$gnu_coreutils-$gnu_coreutils_version.tar.xz"
        "$rust_uutils-$rust_uutils_version.tar.gz::$_url/archive/$rust_uutils_version.tar.gz")
sha512sums=('a6ee2c549140b189e8c1b35e119d4289ec27244ec0ed9da0ac55202f365a7e33778b1dc7c4e64d1669599ff81a8297fe4f5adbcc8a3a2f75c919a43cd4b9bdfa'
            '6130c4d7ca8a31c95a15b0e5e64cc69ff21e4417fe118d04c354a30933379a99334c87416418ca29f5e31f25f689686d83329f971e970c1802aa5de933ad1207')

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
  # build gnu coreutils excluding all except shasum related programs
  cd $gnu_coreutils-$gnu_coreutils_version
  ./configure \
      --prefix=/usr \
      --libexecdir=/usr/lib \
      --with-openssl \
      --enable-no-install-program="arch,base32,base64,basename,basenc,cat,chcon,chgrp,chmod,chown,chroot,cksum,comm,cp,csplit,cut,date,dd,df,dir,dircolors,dirname,du,echo,env,expand,expr,factor,false,fmt,fold,groups,head,hostid,id,install,join,link,ln,logname,ls,mkdir,mkfifo,mknod,mktemp,mv,nice,nl,nohup,nproc,numfmt,od,paste,pathchk,pinky,pr,printenv,printf,ptx,pwd,readlink,realpath,rm,rmdir,runcon,seq,setuidgid,shred,shuf,sleep,sort,split,stat,stdbuf,stty,sum,sync,tac,tail,tee,test,touch,tr,true,truncate,tsort,tty,uname,unexpand,uniq,unlink,users,vdir,wc,who,whoami,yes"
}

package() {
  # install uutils-coreutils, skip the buggy parts
  cd $gnu_coreutils-$rust_uutils_version
  make \
      DESTDIR="$pkgdir" \
      PREFIX=/usr \
      MANDIR=/share/man/man1 \
      PROG_PREFIX= \
      PROFILE=release \
      MULTICALL=y \
      install

  # install gnu coreutils over the uutils-coreutils
  cd $srcdir && cd $gnu_coreutils-$gnu_coreutils_version
  make DESTDIR="$pkgdir" install

  # clean conflicts, arch ships these in other apps
  cd $pkgdir/usr/bin
  rm groups hostname kill more uptime
  rm $pkgdir/usr/share/bash-completion/completions/*
  rm $pkgdir/usr/share/man/man1/{groups.1,hostname.1,kill.1,more.1,uptime.1}
}
