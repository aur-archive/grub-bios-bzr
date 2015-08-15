# Maintainer : Keshav P R <(the.ridikulus.rat) (aatt) (gemmaeiil) (ddoott) (ccoomm)>

_pkgname="grub-bios-bzr"

pkgname="${_pkgname}"          ## Uncomment for grub BZR Main Branch
# pkgname="${_pkgname}-exp"    ## Uncomment for grub BZR Experimental Branch

pkgver="4750"
pkgrel="2"

url="https://www.gnu.org/software/grub/"
arch=('i686' 'x86_64')
license=('GPL3')

makedepends=('bzr' 'rsync' 'xz' 'bdf-unifont' 'ttf-dejavu' 'python' 'autogen' 'texinfo' 'help2man' 'gettext' 'device-mapper' 'fuse')
depends=('xz' 'freetype2' 'device-mapper' 'fuse' 'gettext' 'sh')
optdepends=('libisoburn: provides xorriso for generating grub rescue iso using grub-mkrescue'
            'os-prober: to detect other OSes when generating grub.cfg in BIOS systems')

install="${_pkgname}.install"
backup=('boot/grub/grub.cfg' 'etc/default/grub' 'etc/grub.d/40_custom')

conflicts=('grub-common' 'grub-bios')
provides=('grub-common' 'grub-bios')

source=('grub.default'
        'grub.cfg'
        '60_memtest86+'
        'archlinux_grub_mkconfig_fixes.patch')

sha1sums=('dbf493dec4722feb11f0b5c71ad453a18daf0fc5'
          '76ae862a945a8848e6999adf8ad1847f0f7008b9'
          '46d5c88b0acaa834992de3eaf23168cf2ae29d6f'
          'a2a03ae0255b4b29c6ce6147efdf6a9daba7dcd5')


if [[ "${pkgname}" == "${_pkgname}-exp" ]]; then
	pkgdesc="The GNU GRand Unified Bootloader (2) - i386 PC BIOS - bzr experimental branch with grub-extras"
	provides+=('grub2-bios-bzr-exp')
	
	_bzrtrunk="bzr://bzr.savannah.gnu.org/grub/branches/experimental/"
	# _bzrtrunk="lp:~the-ridikulus-rat/grub/grub-bzr-exp"
	_bzrmod="grub_exp"
else
	pkgdesc="The GNU GRand Unified Bootloader (2) - i386 PC BIOS - bzr mainline branch with grub-extras"
	provides+=('grub2-bios-bzr')
	
	_bzrtrunk="bzr://bzr.savannah.gnu.org/grub/trunk/grub/"
	# _bzrtrunk="lp:grub/grub2"
	_bzrmod="grub"
fi

if [[ "${CARCH}" == 'x86_64' ]]; then
	# makedepends+=('gcc-multilib' 'binutils-multilib')
	
	_EFIEMU="--enable-efiemu"
else
	_EFIEMU="--disable-efiemu"
fi


## grub-extras bzr repo locations

_bzrtrunk_lua="bzr://bzr.savannah.gnu.org/grub-extras/lua/"
# _bzrtrunk_lua="lp:~the-ridikulus-rat/grub/grub2-extras-lua"

_bzrtrunk_ntldr_img="bzr://bzr.savannah.gnu.org/grub-extras/ntldr-img/"
# _bzrtrunk_ntldr_img="lp:~the-ridikulus-rat/grub/grub2-extras-ntldr-img"

_bzrtrunk_915resolution="bzr://bzr.savannah.gnu.org/grub-extras/915resolution/"
# _bzrtrunk_915resolution="lp:~the-ridikulus-rat/grub/grub2-extras-915resolution"


_update_bzr() {
	
	msg "Connecting to BZR server..."
	
	if [[ -d "${srcdir}/${_bzrmod}" ]]; then
		cd "${srcdir}/${_bzrmod}"
		bzr pull "${_bzrtrunk}"
		msg "GRUB BZR Local repository updated."
	else
		cd "${srcdir}/"
		bzr branch "${_bzrtrunk}" "${_bzrmod}"
		msg "GRUB BZR repository cloned."
	fi
	msg "BZR checkout done or server timeout"
	
	if [[ -d "${srcdir}/${_bzrmod}/grub-extras" ]]; then
		cd "${srcdir}/${_bzrmod}/grub-extras/"
		
		if [[ -d "${srcdir}/${_bzrmod}/grub-extras/lua" ]]; then
			cd "${srcdir}/${_bzrmod}/grub-extras/lua"
			bzr pull "${_bzrtrunk_lua}"
			echo
		else
			bzr branch "${_bzrtrunk_lua}" lua
			echo
		fi
		
		if [[ -d "${srcdir}/${_bzrmod}/grub-extras/ntldr-img" ]]; then
			cd "${srcdir}/${_bzrmod}/grub-extras/ntldr-img"
			bzr pull "${_bzrtrunk_ntldr_img}"
			echo
		else
			bzr branch "${_bzrtrunk_ntldr_img}" ntldr-img
			echo
		fi
		
		if [[ -d "${srcdir}/${_bzrmod}/grub-extras/915resolution" ]]; then
			cd "${srcdir}/${_bzrmod}/grub-extras/915resolution"
			bzr pull "${_bzrtrunk_915resolution}"
			echo
		else
			bzr branch "${_bzrtrunk_915resolution}" 915resolution
			echo
		fi
	else
		mkdir -p "${srcdir}/${_bzrmod}/grub-extras/"
		cd "${srcdir}/${_bzrmod}/grub-extras/"
		
		bzr branch "${_bzrtrunk_lua}" lua
		echo
		bzr branch "${_bzrtrunk_ntldr_img}" ntldr-img
		echo
		bzr branch "${_bzrtrunk_915resolution}" 915resolution
		echo
	fi
	
	cd "${srcdir}/${_bzrmod}/"
	rsync -Lrtvz translationproject.org::tp/latest/grub/ "${srcdir}/${_bzrmod}/po" || true
	(cd "${srcdir}/${_bzrmod}/po" && ls *.po | cut -d. -f1 | xargs) > "${srcdir}/${_bzrmod}/po/LINGUAS"
	
}


build() {
	
	_update_bzr
	
	rm -rf "${srcdir}/${_bzrmod}_build" || true
	
	rm -rf "${srcdir}/${_bzrmod}/grub-extras/zfs" || true
	rm -rf "${srcdir}/${_bzrmod}/grub-extras/gpxe" || true
	
	cp -r "${srcdir}/${_bzrmod}" "${srcdir}/${_bzrmod}_build"
	cd "${srcdir}/${_bzrmod}_build"
	
	## Apply Archlinux specific fixes to enable grub-mkconfig detect Arch kernels and initramfs
	patch -Np1 -i "${srcdir}/archlinux_grub_mkconfig_fixes.patch"
	echo
	
	## fix unifont.bdf location so that grub-mkfont can create *.pf2 files
	sed 's|/usr/share/fonts/unifont|/usr/share/fonts/unifont /usr/share/fonts/misc|g' -i "${srcdir}/${_bzrmod}_build/configure.ac"
	
	## fix DejaVuSans.ttf location so that grub-mkfont can create *.pf2 files for starfield theme
	sed 's|/usr/share/fonts/dejavu|/usr/share/fonts/dejavu /usr/share/fonts/TTF|g' -i "${srcdir}/${_bzrmod}_build/configure.ac"
	
    rm -f "${srcdir}/${_bzrmod}_build/po/POTFILES.in" || true
    rm -f "${srcdir}/${_bzrmod}_build/po/POTFILES-shell.in" || true

    find . -iname '*.[ch]' | sort > "${srcdir}/${_bzrmod}_build/po/POTFILES.in"
    find ./util -iname '*.in' | sort | sed '/bash-completion.d/d' > "${srcdir}/${_bzrmod}_build/po/POTFILES-shell.in"

    sed -i 's/grub-dev.texi//g' "${srcdir}/${_bzrmod}_build/docs/Makefile.am" || true

	export GRUB_CONTRIB="${srcdir}/${_bzrmod}_build/grub-extras/"
	
	## Requires python2
	# install -D -m0755 "${srcdir}/${_bzrmod}_build/autogen.sh" "${srcdir}/${_bzrmod}_build/autogen_unmodified.sh"
	# sed 's|python |python2 |g' -i "${srcdir}/${_bzrmod}_build/autogen.sh"
	echo
	
	"${srcdir}/${_bzrmod}_build/autogen.sh"
	echo
	
	mkdir -p "${srcdir}/${_bzrmod}_build/BUILD_BIOS"
	cd "${srcdir}/${_bzrmod}_build/BUILD_BIOS"
	
	CFLAGS="" ../configure \
		--with-platform="pc" \
		--target="i386" \
		--host="${CARCH}-unknown-linux-gnu" \
		"${_EFIEMU}" \
		--enable-mm-debug \
		--enable-nls \
		--enable-device-mapper \
		--enable-cache-stats \
		--enable-grub-mkfont \
		--enable-grub-mount \
		--prefix="/usr" \
		--bindir="/usr/bin" \
		--sbindir="/usr/sbin" \
		--mandir="/usr/share/man" \
		--infodir="/usr/share/info" \
		--datarootdir="/usr/share" \
		--sysconfdir="/etc" \
		--program-prefix="" \
		--with-bootdir="/boot" \
		--with-grubdir="grub" \
		--disable-werror
	echo
	
	CFLAGS="" make
	echo
	
}

package() {
	
	cd "${srcdir}/${_bzrmod}_build/BUILD_BIOS"
	make DESTDIR="${pkgdir}/" bashcompletiondir="/usr/share/bash-completion/completions/" install
	echo
	
	## install /etc/default/grub
	install -D -m0644 "${srcdir}/grub.default" "${pkgdir}/etc/default/grub"
	
	## install grub.cfg for /boot/grub/grub.cfg backup
	install -D -m0644 "${srcdir}/grub.cfg" "${pkgdir}/boot/grub/grub.cfg"
	
	## install memtest config detection
	install -D -m0755 "${srcdir}/60_memtest86+" "${pkgdir}/etc/grub.d/60_memtest86+"
	
}
