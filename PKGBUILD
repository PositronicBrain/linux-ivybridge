# Maintainer: Federico Squartini <federico.squartini@gmail.com>


pkgname=(kernel-ivybridge kernel-ivybridge-headers)
pkgver=3.7.0
_kernver=3.7
_patchkernver=3.7.0
_kernname="${_patchkernver}"-ivybridge
_bfsrel=426
_bfqver=v5r1
_bfqpath="http://algo.ing.unimo.it/people/paolo/disk_sched/patches/${_patchkernver}-${_bfqver}/"
_bfspath="http://ck.kolivas.org/patches/bfs/3.0/${_kernver}/"
_bfspatchname="${_kernver}-sched-bfs-${_bfsrel}.patch"
_bfq1patchname="0001-block-cgroups-kconfig-build-bits-for-BFQ-${_bfqver}-${_kernver}.patch"
_bfq2patchname="0002-block-introduce-the-BFQ-${_bfqver}-I-O-sched-for-${_kernver}.patch"
pkgrel=1
KARCH=x86

arch=('x86_64')
license=('GPL2')
url="http://www.kernel.org"
depends=('coreutils' 'module-init-tools')

source=(http://www.kernel.org/pub/linux/kernel/v3.x/linux-$_kernver.tar.bz2
    ${_bfspath}/${_bfspatchname}
    ${_bfqpath}/${_bfq1patchname}
    ${_bfqpath}/${_bfq2patchname}
    kernelconfig)

md5sums=(5323f3faadd051e83af605a63be5ea2e
    c3af8766b1bce7c8c3c6574a0f6b7a25
    9daa5f662145f91b25b91b9fbeb874d9
    c80954ae588d8c168a0b1ae9ffe84c0e
    8e97330025d6288879afcf514178811f
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
    cp $startdir/kernelconfig ./.config
    make clean
    make gconfig
   # make silentoldconfig
    make || return 1
    cp -vf $srcdir/linux-$_kernver/.config $startdir/kernelconfig.new
}

package_kernel-ivybridge() {
    pkgname=kernel-ivybridge
    pkgdesc="Linux Kernel and modules for Intel Ivybridge platform"
    install=kernel-ivybridge.install
    cd $srcdir/linux-$_kernver

    # install the kernel
    msg "Installing kernel"
    mkdir -p $pkgdir/boot
    cp System.map $pkgdir/boot/System.map."$pkgver"-ivybridge
    cp arch/x86/boot/bzImage $pkgdir/boot/vmlinuz-"$pkgver"-ivybridge
    ln -s vmlinuz-"$pkgver"-ivybridge $pkgdir/boot/vmlinuz-ivybridge
    ln -s System.map."$pkgver"-ivybridge $pkgdir/boot/System.map.ivybridge

    # install our modules
    msg "Installing modules"
    mkdir -p $pkgdir/lib/modules
    mkdir -p $pkgdir/usr/lib

    make INSTALL_MOD_PATH=$pkgdir modules_install || return 1

    rm -f "${pkgdir}"/lib/modules/"$_kernname"/{build,source}
    rm -f "${pkgdir}"/lib/firmware

    # gzip -9 all modules
    msg "Compressing modules"
    find "${pkgdir}" -name '*ko' -exec gzip -9 {} \;

    # move module tree /lib -> /usr/lib
    mv -v "$pkgdir/lib" "$pkgdir/usr"

    # install a helper file for all install scripts
    mkdir -p $pkgdir/usr/share/kernel-ivybridge/
    echo "KERNEL_VERSION='$_kernname'" > $pkgdir/usr/share/kernel-ivybridge/currver

}


package_kernel-ivybridge-headers() {
    pkgname=kernel-ivybridge-headers
    pkgdesc="Header files and scripts to build modules for kernel-ivibridge."
    depends=('kernel-ivybridge')
    KARCH=x86
    install -dm755 "${pkgdir}/usr/lib/modules/${_kernname}"

    cd "${pkgdir}/usr/lib/modules/${_kernname}"
    ln -sf ../../../src/${_kernname} build

    cd ${srcdir}/linux-$_kernver
    install -D -m644 Makefile \
        "${pkgdir}/usr/src/${_kernname}/Makefile"
    install -D -m644 kernel/Makefile \
        "${pkgdir}/usr/src/${_kernname}/kernel/Makefile"
    install -D -m644 .config \
        "${pkgdir}/usr/src/${_kernname}/.config"

    mkdir -p "${pkgdir}/usr/src/${_kernname}/include"

    for i in acpi asm-generic config crypto drm generated linux math-emu \
        media net scsi sound trace uapi video xen; do
        cp -a include/${i} "${pkgdir}/usr/src/${_kernname}/include/"
    done

      # copy arch includes for external modules
    mkdir -p "${pkgdir}/usr/src/${_kernname}/arch/x86"
    cp -a arch/x86/include "${pkgdir}/usr/src/${_kernname}/arch/x86/"

      # copy files necessary for later builds, like nvidia and vmware
    cp Module.symvers "${pkgdir}/usr/src/${_kernname}"
    cp -a scripts "${pkgdir}/usr/src/${_kernname}"

      # fix permissions on scripts dir
    chmod og-w -R "${pkgdir}/usr/src/${_kernname}/scripts"
    mkdir -p "${pkgdir}/usr/src/${_kernname}/.tmp_versions"

    mkdir -p "${pkgdir}/usr/src/${_kernname}/arch/${KARCH}/kernel"

    cp arch/${KARCH}/Makefile "${pkgdir}/usr/src/${_kernname}/arch/${KARCH}/"

    if [ "${CARCH}" = "i686" ]; then
        cp arch/${KARCH}/Makefile_32.cpu "${pkgdir}/usr/src/${_kernname}/arch/${KARCH}/"
    fi

    cp arch/${KARCH}/kernel/asm-offsets.s "${pkgdir}/usr/src/${_kernname}/arch/${KARCH}/kernel/"

      # add docbook makefile
    install -D -m644 Documentation/DocBook/Makefile \
        "${pkgdir}/usr/src/${_kernname}/Documentation/DocBook/Makefile"

      # add dm headers
    mkdir -p "${pkgdir}/usr/src/${_kernname}/drivers/md"
    cp drivers/md/*.h "${pkgdir}/usr/src/${_kernname}/drivers/md"

      # add inotify.h
    mkdir -p "${pkgdir}/usr/src/${_kernname}/include/linux"
    cp include/linux/inotify.h "${pkgdir}/usr/src/${_kernname}/include/linux/"

      # copy in Kconfig files
    for i in `find . -name "Kconfig*"`; do
        mkdir -p "${pkgdir}"/usr/src/${_kernname}/`echo ${i} | sed 's|/Kconfig.*||'`
        cp ${i} "${pkgdir}/usr/src/${_kernname}/${i}"
    done

    chown -R root.root "${pkgdir}/usr/src/${_kernname}"
    find "${pkgdir}/usr/src/${_kernname}" -type d -exec chmod 755 {} \;

      # strip scripts directory
    find "${pkgdir}/usr/src/${_kernname}/scripts" -type f -perm -u+w 2>/dev/null | while read binary ; do
        case "$(file -bi "${binary}")" in
            *application/x-sharedlib*) # Libraries (.so)
                /usr/bin/strip ${STRIP_SHARED} "${binary}";;
            *application/x-archive*) # Libraries (.a)
                /usr/bin/strip ${STRIP_STATIC} "${binary}";;
            *application/x-executable*) # Binaries
                /usr/bin/strip ${STRIP_BINARIES} "${binary}";;
        esac
    done

# remove unneeded architectures
    rm -rf "${pkgdir}"/usr/src/${_kernname}/arch/{alpha,arm,arm26,avr32,\
                        blackfin,c6x,cris,frv,h8300,hexagon,ia64,m32r,m68k,\
                        m68knommu,mips,microblaze,mn10300,openrisc,parisc,\
                        powerpc,ppc,s390,score,sh,sh64,sparc,sparc64,tile,\
                        unicore32,um,v850,xtensa}
}
