# Maintainer: Federico Squartini <federico.squartini@googlemail.com>

pkgname=kernel-ivybridge
pkgver=3.7.0
_kernver=3.7
_patchkernver=3.7.0
_bfsrel=426
_bfqver=v5r1
_bfqpath="http://algo.ing.unimo.it/people/paolo/disk_sched/patches/${_patchkernver}-${_bfqver}/"
_bfspath="http://ck.kolivas.org/patches/bfs/3.0/${_kernver}/"
_bfspatchname="${_kernver}-sched-bfs-${_bfsrel}.patch"
_bfq1patchname="0001-block-cgroups-kconfig-build-bits-for-BFQ-${_bfqver}-${_kernver}.patch"
_bfq2patchname="0002-block-introduce-the-BFQ-${_bfqver}-I-O-sched-for-${_kernver}.patch"
pkgrel=1
pkgdesc="Linux Kernel and modules for Intel Ivybridge platform"
arch=('x86_64')
license=('GPL2')
url="http://www.kernel.org"
depends=('coreutils' 'module-init-tools')
install=kernel-ivybridge.install
source=(http://www.kernel.org/pub/linux/kernel/v3.x/linux-$_kernver.tar.bz2
        ${_bfspath}/${_bfspatchname}
        ${_bfqpath}/${_bfq1patchname}
        ${_bfqpath}/${_bfq2patchname}
        kernelconfig)

md5sums=(5323f3faadd051e83af605a63be5ea2e
         c3af8766b1bce7c8c3c6574a0f6b7a25
         9daa5f662145f91b25b91b9fbeb874d9
         c80954ae588d8c168a0b1ae9ffe84c0e
         1b050cbc285fa8cbfd166d0274fb6397
)

build() {
   # unset since our build machine may differ from eee
   unset CFLAGS CXXFLAGS

   # get into the linux source directory and start some magic
   cd $srcdir/linux-$_kernver

   #msg "Patching source with BFS patch"
   patch -Np1 -i $srcdir/${_bfspatchname}

   # Patch source for BFQ patches

   msg "Patching source with BFQ patches"
   patch -Np1 -i  $srcdir/${_bfq1patchname}
   patch -Np1 -i  $srcdir/${_bfq2patchname}

   # don't run depmod on 'make install'. We'll do this ourselves in packaging
   sed -i '2iexit 0' scripts/depmod.sh

   # load configuration and build kernel + modules
   cp ../kernelconfig ./.config
   make clean
   make gconfig
   # # make silentoldconfig
   make || return 1
   cp ./.config ../../kernelconfig.new
}

package() {
  # install our modules
  cd $srcdir/linux-$_kernver

  mkdir -p $pkgdir/{lib/modules,boot}
  mkdir -p $pkgdir/usr/lib

  make INSTALL_MOD_PATH=$pkgdir modules_install || return 1

  # install the kernel
  cp System.map $pkgdir/boot/System.map."$pkgver"-ivybridge
  cp arch/x86/boot/bzImage $pkgdir/boot/vmlinuz-"$pkgver"-ivybridge
  ln -s vmlinuz-"$pkgver"-ivybridge $pkgdir/boot/vmlinuz-ivybridge
  ln -s System.map."$pkgver"-ivybridge $pkgdir/boot/System.map.ivybridge
  #
  rm -f "${pkgdir}"/lib/modules/$pkgver/{build,source}
  rm -f "${pkgdir}"/lib/firmware
  # gzip -9 all modules
  echo "Compressing modules\n"
  find "${pkgdir}" -name '*ko' -exec gzip -9 {} \;

  # move module tree /lib -> /usr/lib
  mv -v "$pkgdir/lib" "$pkgdir/usr"

  # install a helper file for all install scripts
  mkdir -p $pkgdir/usr/share/kernel-ivybridge/
  echo "KERNEL_VERSION='$_patchkernver-ivybridge'" > $pkgdir/usr/share/kernel-ivybridge/currver
}
