# Contributor: graysky <graysky AT archlinux DOT us>
# Contributor: Tobias Powalowski <tpowa@archlinux.org>
# Contributor: Thomas Baechler <thomas@archlinux.org>

# Set these variables to ANYTHING that is not null (y or hello or 2 or "I like icecream") to enable them
#
_pstates_pat=   # Enable Haswell support for the new Intel pstate drive
_makenconfig=   # Tweak kernel options prior to a build via nconfig
_localmodcfg=y   # Compile ONLY probed modules
_use_current=y   # Use the current kernel's .config file
_BFQ_enable_=y   # Enable BFQ as the default I/O scheduler
_NUMAdisable=y  # Disable NUMA in kernel config
_1k_HZ_ticks=y  #Enable 1000Hz tickrate

pkgname=linux-ck
true && pkgname=(linux-ck linux-ck-headers)
_kernelname=-harfmix
_srcname=linux-3.11
pkgver=3.11.1
pkgrel=2
arch=('i686' 'x86_64')
url="https://wiki.archlinux.org/index.php/Linux-ck"
license=('GPL2')
makedepends=('kmod' 'inetutils' 'bc')
options=('!strip')
_ckpatchversion=1
_ckpatchname="patch-3.11-ck${_ckpatchversion}"
_gcc_patch="kernel-311-gcc48-1.patch"
_kbase=3.11
_kpatch=3.11.0
_bfqversion=v6r2
_bfqpath="http://algo.ing.unimo.it/people/paolo/disk_sched/patches/$_kpatch-$_bfqversion"
_kpatchsum=$(sha256sum patch-$pkgver.xz | awk '{print $1}')
_kbasesum=$(sha256sum $_srcname.tar.xz | awk '{print $1}')
_cksum=$(sha256sum ${_ckpatchname}.bz2 | awk '{print $1}')
_gccsum=$(sha256sum $_gcc_patch.gz | awk '{print $1}')
sed -i "55s/.*/\'$_kbasesum\'/g" PKGBUILD
sed -i "56s/.*/\'$_kpatchsum\'/g" PKGBUILD
sed -i "57s/.*/\'$_cksum\'/g" PKGBUILD
sed -i "58s/.*/\'$_gccsum\'/g" PKGBUILD
source=("http://www.kernel.org/pub/linux/kernel/v3.x/${_srcname}.tar.xz"
"http://www.kernel.org/pub/linux/kernel/v3.x/patch-${pkgver}.xz"
"http://ck.kolivas.org/patches/3.0/$_kbase/$_kbase-ck${_ckpatchversion}/${_ckpatchname}.bz2"
"http://repo-ck.com/source/gcc_patch/${_gcc_patch}.gz"
'enable_haswell_pstate_driver.patch'
'linux-ck.preset'
'change-default-console-loglevel.patch'
'config' 'config.x86_64'
"${_bfqpath}/0001-block-cgroups-kconfig-build-bits-for-BFQ-$_bfqversion-$_kbase.patch"
"${_bfqpath}/0002-block-introduce-the-BFQ-$_bfqversion-I-O-sched-for-$_kbase.patch"
"${_bfqpath}/0003-block-bfq-add-Early-Queue-Merge-EQM-to-BFQ-$_bfqversion-for-$_kpatch.patch"
"https://bld.googlecode.com/files/BLD-3.11.0.patch"
"https://raw.github.com/sergiuniculescu/configs/master/kernel%20patch/uksm/uksm-0.1.2.2-for-v3.10.patch")
sha256sums=(
''
''
''
''
'd7fada52453d12a24af9634024c36792697f97ce0bc6552939cd7b2344d00cd9'
'c2cf8cc2600502de348f3dc3aae9a3bde5486759db15cb8a93df7aa35bd6e7da'
'56bd99e54429a25a144f2d221718b67f516344ffd518fd7dcdd752206ec5be69'
'059a8511042a188d283057d3fb27077f3833d987a1b2e3ef4c5bed065184acb3'
'e6fe236c96aff5f16dea24ef70fdc36fdd016ee1d63887f35ae698c31cc8a77f'
'10d14a0adc3189b3d2cde5c2d1f94734d4f19744c17ef219d75ed8874969b125'
'526ac0fa67731e4d60418a8551083a1b8bf2cf3317cac95ad59bda75d342dfd2'
'5dcbf25a92a597100a7990f4844224f6a711de2a0ca2aaa7d40cc06ea0ef2cc8'
'e5604a77f0fc0103bda8aeced81f0a1ed365d3892b07a63eb9a2a3d48871daeb'
'54f94f5f5a7ba560543d77a182def02a06638ea613a60b97065f76370f3beb40'
)

prepare() {
	cd "${_srcname}"

	# add upstream patch
	#msg "Patching source to v$pkgver"
	patch -p1 -i "${srcdir}/patch-${pkgver}"

	# set DEFAULT_CONSOLE_LOGLEVEL to 4
	#patch -Np1 -i "${srcdir}/change-default-console-loglevel.patch"
  
	### Patch source with ck patchset with BFS
	# Fix double name in EXTRAVERSION
	sed -i -re "s/^(.EXTRAVERSION).*$/\1 = /" "${srcdir}/${_ckpatchname}"
	msg "Patching source with ck1 including BFS v0.440"
	patch -Np1 -i "${srcdir}/${_ckpatchname}"

    ### Patch source to enable pstate drivers with ivybridge CPUs
	### see https://plus.google.com/117091380454742934025/posts/2vEekAsG2QT
	if [ -n "$_pstates_pat" ]; then
		msg "Patching source to enable Intel Pstate driver for Haswell CPUs"
		patch -Np1 -i "${srcdir}/enable_haswell_pstate_driver.patch"
	fi
	

	### Patch source to enable more gcc CPU optimizatons via the make nconfig
	patch -Np1 -i "${srcdir}/${_gcc_patch}"

	msg "Patching source with BFQ patches"
	for p in $(ls ${srcdir}/000{1,2,3}-block*.patch); do
		patch -Np1 -i "$p"
	done

	### BLD patch ###
	patch -Np1 -i "${srcdir}/BLD-3.11.0.patch"

	### UKSM ###
	patch -Np1 -i "${srcdir}/uksm-0.1.2.2-for-v3.10.patch"

	### Clean tree and copy ARCH config over
	msg "Running make mrproper to clean source tree"
	make mrproper

	if [ "${CARCH}" = "x86_64" ]; then
		cat "${srcdir}/config.x86_64" > ./.config
	else
		cat "${srcdir}/config" > ./.config
	fi

    ### Optionally set tickrate to 1000 to avoid suspend issues as reported here:
	# http://ck-hack.blogspot.com/2013/09/bfs-0441-311-ck1.html?showComment=1379234249615#c4156123736313039413
	if [ -n "$_1k_HZ_ticks" ]; then
	msg "Setting tick rate to 1k..."
		sed -i -e 's/^CONFIG_HZ_300=y/# CONFIG_HZ_300 is not set/' \
			-i -e 's/^# CONFIG_HZ_1000 is not set/CONFIG_HZ_1000=y/' \
			-i -e 's/^CONFIG_HZ=300/CONFIG_HZ=1000/' .config
	fi

	### Optionally use running kernel's config
	# code originally by nous; http://aur.archlinux.org/packages.php?ID=40191
	if [ -n "$_use_current" ]; then
		if [[ -s /proc/config.gz ]]; then
			msg "Extracting config from /proc/config.gz..."
			# modprobe configs
			zcat /proc/config.gz > ./.config
		else
			warning "You kernel was not compiled with IKCONFIG_PROC!"
			warning "You cannot read the current config!"
			warning "Aborting!"
			exit
		fi
	fi

	if [ "${_kernelname}" != "" ]; then
		sed -i "s|CONFIG_LOCALVERSION=.*|CONFIG_LOCALVERSION=\"${_kernelname}\"|g" ./.config
		sed -i "s|CONFIG_LOCALVERSION_AUTO=.*|CONFIG_LOCALVERSION_AUTO=n|" ./.config
	fi
	
	### BFQ to be compiled in but not enabled
	sed -i -e s'/CONFIG_CFQ_GROUP_IOSCHED=y/CONFIG_CFQ_GROUP_IOSCHED=y\nCONFIG_IOSCHED_BFQ=y\nCONFIG_CGROUP_BFQIO=y/' \
		-i -e s'/CONFIG_DEFAULT_CFQ=y/CONFIG_DEFAULT_CFQ=y\n# CONFIG_DEFAULT_BFQ is not set/' ./.config

	### Optionally enable BFQ as the default io scheduler
	if [ -n "$_BFQ_enable_" ]; then
		sed -i -e '/CONFIG_DEFAULT_IOSCHED/ s,cfq,bfq,' \
		-i -e s'/CONFIG_DEFAULT_CFQ=y/# CONFIG_DEFAULT_CFQ is not set\nCONFIG_DEFAULT_BFQ=y/' ./.config
	fi

	# disable NUMA since 99.9% of users do not have multiple CPUs but do have multiple cores in one CPU
	# see, https://bugs.archlinux.org/task/31187
	if [ -n "$_NUMAdisable" ]; then
		if [ "${CARCH}" = "x86_64" ]; then
			sed -i -e 's/CONFIG_NUMA=y/# CONFIG_NUMA is not set/' \
				-i -e '/CONFIG_AMD_NUMA=y/d' \
				-i -e '/CONFIG_X86_64_ACPI_NUMA=y/d' \
				-i -e '/CONFIG_NODES_SPAN_OTHER_NODES=y/d' \
				-i -e '/# CONFIG_NUMA_EMU is not set/d' \
				-i -e '/CONFIG_NODES_SHIFT=6/d' \
				-i -e '/CONFIG_NEED_MULTIPLE_NODES=y/d' \
				-i -e '/# CONFIG_MOVABLE_NODE is not set/d' \
				-i -e '/CONFIG_USE_PERCPU_NUMA_NODE_ID=y/d' \
				-i -e '/CONFIG_ACPI_NUMA=y/d' ./.config
		fi
	fi

	# set extraversion to pkgrel
	sed -ri "s|^(EXTRAVERSION =).*|\1 -${pkgrel}|" Makefile

	# don't run depmod on 'make install'. We'll do this ourselves in packaging
	sed -i '2iexit 0' scripts/depmod.sh
}

build() {
	cd "${_srcname}"

	# get kernel version
	msg "Running make prepare for you to enable patched options of your choosing"
	make prepare

	### Optionally load needed modules for the make localmodconfig
	# See http://aur.archlinux.org/packages.php?ID=41689
	if [ -n "$_localmodcfg" ]; then
		msg "If you have modprobe_db installed, running it in recall mode now"
		if [ -e /usr/bin/modprobed_db ]; then
			[[ ! -x /usr/bin/sudo ]] && echo "Cannot call modprobe with sudo.  Install via pacman -S sudo and configure to work with this user." && exit 1
			sudo /usr/bin/modprobed_db recall
		fi
		msg "Running Steven Rostedt's make localmodconfig now"
		make localmodconfig
	fi

	if [ -n "$_makenconfig" ]; then
		msg "Running make nconfig"
		make nconfig
	fi

  # save configuration for later reuse
  if [ "${CARCH}" = "x86_64" ]; then
    cat .config > "${startdir}/config.x86_64.last"
	sed -e 's/CONFIG_GENERIC_CPU=y/# CONFIG_GENERIC_CPU is not set/' -e 's/# CONFIG_MCOREAVX2 is not set/CONFIG_MCOREAVX2=y/' \
-e 's/CONFIG_X86_WP_WORKS_OK=y/CONFIG_X86_WP_WORKS_OK=y\nCONFIG_X86_INTEL_USERCOPY=y\nCONFIG_X86_USE_PPRO_CHECKSUM=y\nCONFIG_X86_P6_NOP=y/' .config > .config.tmp
	mv -f .config.tmp .config
  else
    cat .config > "${startdir}/config.last"
  fi

	msg "Running make bzImage and modules"
	make ${MAKEFLAGS} LOCALVERSION= bzImage modules
}

package_linux-ck() {
	_Kpkgdesc='Linux Kernel and modules with the ck1 patchset featuring the Brain Fuck Scheduler v0.440.'
	pkgdesc="${_Kpkgdesc}"
	depends=('coreutils' 'linux-firmware' 'mkinitcpio>=0.7')
	#optdepends=('crda: to set the correct wireless channels of your country' 'lirc-ck: Linux Infrared Remote Control kernel modules for linux-ck' 'nvidia-ck: nVidia drivers for linux-ck' 
#'nvidia-beta-ck: nVidia beta drivers for linux-ck' 'modprobed_db: Keeps track of EVERY kernel module that has ever been probed - useful for those of us who make localmodconfig')
	provides=("linux=${pkgver}")
	conflicts=('kernel26-ck' 'linux-ck-corex' 'linux-ck-p4' 'linux-ck-pentm' 'linux-ck-atom' 'linux-ck-core2' 'linux-ck-nehalem' 'linux-ck-sandybridge' 'linux-ck-ivybridge' 'linux-ck-haswell' 'linux-ck-kx' 'linux-ck-k10' 'linux-ck-barcelona' 'linux-ck-bulldozer' 'linux-ck-piledriver')
	replaces=('kernel26-ck')
	backup=("etc/mkinitcpio.d/linux-ck.preset")
	install=linux-ck.install
	#groups=('ck-generic')

	cd "${_srcname}"

	KARCH=x86

	# get kernel version
	_kernver="$(make LOCALVERSION= kernelrelease)"
	_basekernel=${_kernver%%-*}
	_basekernel=${_basekernel%.*}

	mkdir -p "${pkgdir}"/{lib/modules,lib/firmware,boot}
	make LOCALVERSION= INSTALL_MOD_PATH="${pkgdir}" modules_install
	cp arch/$KARCH/boot/bzImage "${pkgdir}/boot/vmlinuz-linux-ck"

	# add vmlinux
	install -D -m644 vmlinux "${pkgdir}/usr/src/linux-${_kernver}/vmlinux"

	# set correct depmod command for install
	cp -f "${startdir}/${install}" "${startdir}/${install}.pkg"
	true && install=${install}.pkg

	sed \
		-e  "s/KERNEL_NAME=.*/KERNEL_NAME=-ck/g" \
		-e  "s/KERNEL_VERSION=.*/KERNEL_VERSION=${_kernver}/g" \
		-i "${startdir}/${install}"

	# install mkinitcpio preset file for kernel
	install -D -m644 "${srcdir}/linux-ck.preset" "${pkgdir}/etc/mkinitcpio.d/linux-ck.preset"
	sed \
		-e "1s|'linux.*'|'linux-ck'|" \
		-e "s|ALL_kver=.*|ALL_kver=\"/boot/vmlinuz-linux-ck\"|" \
		-e "s|default_image=.*|default_image=\"/boot/initramfs-linux-ck.img\"|" \
		-e "s|fallback_image=.*|fallback_image=\"/boot/initramfs-linux-ck-fallback.img\"|" \
		-i "${pkgdir}/etc/mkinitcpio.d/linux-ck.preset"

	# remove build and source links
	rm -f "${pkgdir}"/lib/modules/${_kernver}/{source,build}
	# remove the firmware
	rm -rf "${pkgdir}/lib/firmware"
	# gzip -9 all modules to save 100MB of space
	find "${pkgdir}" -name '*.ko' -exec gzip -9 {} \;
	# make room for external modules
	ln -s "../extramodules-${_basekernel}${_kernelname:ck}" "${pkgdir}/lib/modules/${_kernver}/extramodules"
	# add real version for building modules and running depmod from post_install/upgrade
	mkdir -p "${pkgdir}/lib/modules/extramodules-${_basekernel}${_kernelname:ck}"
	echo "${_kernver}" > "${pkgdir}/lib/modules/extramodules-${_basekernel}${_kernelname:ck}/version"

	# Now we call depmod...
	depmod -b "$pkgdir" -F System.map "$_kernver"

	# move module tree /lib -> /usr/lib
	mv "$pkgdir/lib" "$pkgdir/usr"
}

package_linux-ck-headers() {
	_Hpkgdesc='Header files and scripts to build modules for linux-ck.'
	pkgdesc="${_Hpkgdesc}"
	depends=('linux-ck') # added to keep kernel and headers packages matched
	provides=("linux-ck-headers=${pkgver}" "linux-headers=${pkgver}")
	conflicts=('kernel26-ck-headers' 'linux-ck-corex-headers' 'linux-ck-p4-headers' 'linux-ck-pentm-headers' 'linux-ck-atom-headers' 'linux-ck-core2-headers' 'linux-ck-nehalem-headers' 'linux-ck-sandybridge-headers' 'linux-ck-ivybridge-headers' 'linux-ck-haswell-headers' 'linux-ck-kx-headers' 'linux-ck-k10-headers' 'linux-ck-barcelona-headers' 'linux-ck-bulldozer-headers' 'linux-ck-piledriver-headers')
	replaces=('kernel26-ck-headers')
	#groups=('ck-generic')

	install -dm755 "${pkgdir}/usr/lib/modules/${_kernver}"

	cd "${pkgdir}/usr/lib/modules/${_kernver}"
	ln -sf ../../../src/linux-${_kernver} build

	cd "${srcdir}/${_srcname}"
	install -D -m644 Makefile \
		"${pkgdir}/usr/src/linux-${_kernver}/Makefile"
	install -D -m644 kernel/Makefile \
		"${pkgdir}/usr/src/linux-${_kernver}/kernel/Makefile"
	install -D -m644 .config \
		"${pkgdir}/usr/src/linux-${_kernver}/.config"

	mkdir -p "${pkgdir}/usr/src/linux-${_kernver}/include"

	for i in acpi asm-generic config crypto drm generated keys linux math-emu \
		media net pcmcia scsi sound trace uapi video xen; do
		cp -a include/${i} "${pkgdir}/usr/src/linux-${_kernver}/include/"
	done

	# copy arch includes for external modules
	mkdir -p "${pkgdir}/usr/src/linux-${_kernver}/arch/x86"
	cp -a arch/x86/include "${pkgdir}/usr/src/linux-${_kernver}/arch/x86/"

	# copy files necessary for later builds, like nvidia and vmware
	cp Module.symvers "${pkgdir}/usr/src/linux-${_kernver}"
	cp -a scripts "${pkgdir}/usr/src/linux-${_kernver}"

	# fix permissions on scripts dir
	chmod og-w -R "${pkgdir}/usr/src/linux-${_kernver}/scripts"
	mkdir -p "${pkgdir}/usr/src/linux-${_kernver}/.tmp_versions"

	mkdir -p "${pkgdir}/usr/src/linux-${_kernver}/arch/${KARCH}/kernel"

	cp arch/${KARCH}/Makefile "${pkgdir}/usr/src/linux-${_kernver}/arch/${KARCH}/"

	if [ "${CARCH}" = "i686" ]; then
		cp arch/${KARCH}/Makefile_32.cpu "${pkgdir}/usr/src/linux-${_kernver}/arch/${KARCH}/"
	fi

	cp arch/${KARCH}/kernel/asm-offsets.s "${pkgdir}/usr/src/linux-${_kernver}/arch/${KARCH}/kernel/"

	# add headers for lirc package
	# pci
	for i in bt8xx cx88 saa7134; do
		mkdir -p "${pkgdir}/usr/src/linux-${_kernver}/drivers/media/pci/${i}"
		cp -a drivers/media/pci/${i}/*.h "${pkgdir}/usr/src/linux-${_kernver}/drivers/media/pci/${i}"
	done
	# usb
	for i in cpia2 em28xx pwc sn9c102; do
		mkdir -p "${pkgdir}/usr/src/linux-${_kernver}/drivers/media/usb/${i}"
		cp -a drivers/media/usb/${i}/*.h "${pkgdir}/usr/src/linux-${_kernver}/drivers/media/usb/${i}"
	done
	# i2c
	mkdir -p "${pkgdir}/usr/src/linux-${_kernver}/drivers/media/i2c"
	cp drivers/media/i2c/*.h  "${pkgdir}/usr/src/linux-${_kernver}/drivers/media/i2c/"
	for i in cx25840; do
		mkdir -p "${pkgdir}/usr/src/linux-${_kernver}/drivers/media/i2c/${i}"
		cp -a drivers/media/i2c/${i}/*.h "${pkgdir}/usr/src/linux-${_kernver}/drivers/media/i2c/${i}"
	done

	# add docbook makefile
	install -D -m644 Documentation/DocBook/Makefile \
		"${pkgdir}/usr/src/linux-${_kernver}/Documentation/DocBook/Makefile"

	# add dm headers
	mkdir -p "${pkgdir}/usr/src/linux-${_kernver}/drivers/md"
	cp drivers/md/*.h "${pkgdir}/usr/src/linux-${_kernver}/drivers/md"

	# add inotify.h
	mkdir -p "${pkgdir}/usr/src/linux-${_kernver}/include/linux"
	cp include/linux/inotify.h "${pkgdir}/usr/src/linux-${_kernver}/include/linux/"

	# add wireless headers
	mkdir -p "${pkgdir}/usr/src/linux-${_kernver}/net/mac80211/"
	cp net/mac80211/*.h "${pkgdir}/usr/src/linux-${_kernver}/net/mac80211/"

	# add dvb headers for external modules
	# in reference to:
	# http://bugs.archlinux.org/task/9912
	mkdir -p "${pkgdir}/usr/src/linux-${_kernver}/drivers/media/dvb-core"
	cp drivers/media/dvb-core/*.h "${pkgdir}/usr/src/linux-${_kernver}/drivers/media/dvb-core/"
	# and...
	# http://bugs.archlinux.org/task/11194
	mkdir -p "${pkgdir}/usr/src/linux-${_kernver}/include/config/dvb/"
	[[ -n $(ls include/config/dvb/*.h &>/dev/null) ]] && cp include/config/dvb/*.h "${pkgdir}/usr/src/linux-${_kernver}/include/config/dvb/"

	# add dvb headers for http://mcentral.de/hg/~mrec/em28xx-new
  # in reference to:
	# http://bugs.archlinux.org/task/13146
	mkdir -p "${pkgdir}/usr/src/linux-${_kernver}/drivers/media/dvb-frontends/"
	cp drivers/media/dvb-frontends/lgdt330x.h "${pkgdir}/usr/src/linux-${_kernver}/drivers/media/dvb-frontends/"
	cp drivers/media/i2c/msp3400-driver.h "${pkgdir}/usr/src/linux-${_kernver}/drivers/media/i2c/"

	# add dvb headers
	# in reference to:
	# http://bugs.archlinux.org/task/20402
	mkdir -p "${pkgdir}/usr/src/linux-${_kernver}/drivers/media/usb/dvb-usb"
	cp drivers/media/usb/dvb-usb/*.h "${pkgdir}/usr/src/linux-${_kernver}/drivers/media/usb/dvb-usb/"
	mkdir -p "${pkgdir}/usr/src/linux-${_kernver}/drivers/media/dvb-frontends"
	cp drivers/media/dvb-frontends/*.h "${pkgdir}/usr/src/linux-${_kernver}/drivers/media/dvb-frontends/"
	mkdir -p "${pkgdir}/usr/src/linux-${_kernver}/drivers/media/tuners"
	cp drivers/media/tuners/*.h "${pkgdir}/usr/src/linux-${_kernver}/drivers/media/tuners/"

	# add xfs and shmem for aufs building
	mkdir -p "${pkgdir}/usr/src/linux-${_kernver}/fs/xfs"
	mkdir -p "${pkgdir}/usr/src/linux-${_kernver}/mm"
	cp fs/xfs/xfs_sb.h "${pkgdir}/usr/src/linux-${_kernver}/fs/xfs/xfs_sb.h"

	# copy in Kconfig files
	for i in `find . -name "Kconfig*"`; do
		mkdir -p "${pkgdir}"/usr/src/linux-${_kernver}/`echo ${i} | sed 's|/Kconfig.*||'`
		cp ${i} "${pkgdir}/usr/src/linux-${_kernver}/${i}"
	done

	chown -R root.root "${pkgdir}/usr/src/linux-${_kernver}"
	find "${pkgdir}/usr/src/linux-${_kernver}" -type d -exec chmod 755 {} \;

	# strip scripts directory
	find "${pkgdir}/usr/src/linux-${_kernver}/scripts" -type f -perm -u+w 2>/dev/null | while read binary ; do
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
	rm -rf "${pkgdir}"/usr/src/linux-${_kernver}/arch/{alpha,arc,arm,arm26,arm64,avr32,blackfin,c6x,cris,frv,h8300,hexagon,ia64,m32r,m68k,m68knommu,metag,mips,microblaze,mn10300,openrisc,parisc,powerpc,ppc,s390,score,sh,sh64,sparc,sparc64,tile,unicore32,um,v850,xtensa}
}

# Global pkgdesc and depends are here so that they will be picked up by AUR
pkgdesc='Linux Kernel and modules with the ck1 patchset featuring the Brain Fuck Scheduler v0.440.'
