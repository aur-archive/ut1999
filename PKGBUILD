# Contributor: quantax -- contact via Arch Linux forum or AUR
# Maintainer: Ben R <thebenj88 *AT* gmail *DOT* com>

pkgname=ut1999
pkgver=451
pkgrel=2
pkgdesc="The classic Unreal Tournament form 1999. Retail CD or DVD required."
arch=('i686' 'x86_64')
url="http://www.unrealtournament2004.com/utgoty/"
license=('custom')
groups=(ut1999-goty)
changelog=${pkgname}.changelog
if [ "$CARCH" = "x86_64" ]; then
    depends=('lib32-alsa-lib' 'lib32-libgl' 'lib32-sdl' 'lib32-openal')
    optdepends=('lib32-nvidia-utils: Accelerated 3D with the NVIDIA binary blob video driver'
                'lib32-catalyst-utils: Accelerated 3D with the AMD/ATI binary blob video driver')
else
    depends=('libgl' 'sdl' 'openal' 'alsa-lib')
fi
makedepends=(makepkg-lib-unreal unshield)
replaces=(ut ut-server)
source=(ut436.run::'http://www.liflg.org/?what=dl&catid=6&gameid=51&filename=unreal.tournament_436-multilanguage.run' \
        ut436goty.run::'http://www.liflg.org/?what=dl&catid=6&gameid=51&filename=unreal.tournament_436-multilanguage.goty.run' \
        http://www.utpg.org/patches/UTPGPatch451.tar.bz2 \
        ut.desktop \
        disk.list \
        utpg.list)
md5sums=('726aede817997a2aefccb8c20601d760' \
         '7012dc6caaa9453dcf8951474556912a' \
         '77a735a78b1eb819042338859900b83b' \
         '0c94a4c1fda89a53a905756d4c15c5a9' \
         '7fbe728f7cef23d53bdfe6be17b7129a' \
         '72efce99d512b1a71587c2c127dccd06')
noextract=(UTPGPatch451.tar.bz2)

# You can uncomment and set these two variables in order to override the auto
# detection done in build() by _detect_cdpath() and _detect_cdversion().
#_cdpath=""    # path to your mounted UT CD or DVD
#_cdversion="" # "default" or "anthology"

# Detect the mount point of the install medium.
_detect_cdpath() {
    echo "Searching for mount point of install medium... "

    for mountpoint in $(egrep "(iso9660|udf)" /etc/mtab \
            | awk '{print $2}'); do
        if [ -f "${mountpoint}/SYSTEM/UnrealTournament.exe" ] \
                || [ -f "${mountpoint}/Disk1/data1.hdr" ]; then
            _cdpath="${mountpoint}"
            break
        fi
    done

    if [ -z "${_cdpath}" ]; then
        /bin/cat << __EOF__ >&2
    No mounted valid Unreal Tournament CD or Unreal Anthology
    DVD has been detected while scanning all "iso9660"
    and all "udf" filesystems in "/etc/mtab" for the file
    "SYSTEM/UnrealTournament.exe" or the file "Disk1/data1.hdr".
    Make sure you mounted the right disk correctly.  If it still
    does not work you can try setting the "_cdpath" and/or the
    "_cdversion" variable in this PKGBUILD to your mount point and
    your version of UT manually.
__EOF__
    else
        echo "    ${_cdpath} looks promising."
    fi
}

# Determine which method should be used for extracting the files from the
# install medium.
_detect_cdversion() {
    echo "Determining install method... "

    if [ -f "${_cdpath}/SYSTEM/UnrealTournament.exe" ]; then
        _cdversion="default"
    elif [ -f "${_cdpath}/Disk1/data1.hdr" ]; then
        _cdversion="anthology"
    else
        echo "Could not determine _cdversion." >&2
    fi
    echo "    Using \"${_cdversion}\" method."
}

# Install files from most UT99 CDs.
_build_default() {
    echo "Extracting files from ${_cdpath}..."
    cd "${srcdir}"

    _unreal_install_files "${_cdpath}" "${pkgdir}/opt/ut" "*./System400/.*" \
            < disk.list

    _install_patches

    echo "Decompressing maps from ${_cdpath}..."
    grep "Maps/" disk.list | sed -e "s/$/\.uz/" \
            | _unreal_decompress_files "${_cdpath}" "${pkgdir}/opt/ut" \
    grep "Maps/" disk.list \
            | _unreal_move_files "${pkgdir}/opt/ut/System" "${pkgdir}/opt/ut" \
    rm -f -- "${pkgdir}/opt/ut/System/ucc.log"
}

# Install files from the Unreal Anthology DVD.
_build_anthology() {
    echo "Extracting files from ${_cdpath}..."
    cd "${srcdir}"
	
	
    ln -fs -- "${_cdpath}/Disk*/data*" -t .
    unshield x -g 3_UnrealTournament_Help -d dvd data1.hdr
    unshield x -g 3_UnrealTournament_Maps -d dvd data1.hdr
    unshield x -g 3_UnrealTournament_Music -d dvd data1.hdr
    unshield x -g 3_UnrealTournament_Sounds_All -d dvd data1.hdr \
    unshield x -g 3_UnrealTournament_Sounds_English -d dvd data1.hdr \
    unshield x -g 3_UnrealTournament_System_All -d dvd data1.hdr \
    unshield x -g 3_UnrealTournament_System_English -d dvd data1.hdr \
    unshield x -g 3_UnrealTournament_Textures -d dvd data1.hdr \

    _unreal_move_files dvd "${pkgdir}/opt/ut" < disk.list
    _install_patches
}

# Add files for running UT on Linux, apply the patches shipped by Loki and add
# some third party fixes.
_install_patches() {
    echo "Adding Loki's Linux runtime files..."
    cd "${srcdir}"

    sh ./ut436.run --tar xfC 436
    sh ./ut436goty.run --tar xfC 436goty

    cd 436goty
    install --mode=644 -D -- ut.xpm "${pkgdir}/usr/share/pixmaps/ut.xpm"
    install --mode=644 -- README "${pkgdir}/opt/ut/Help/README"
    install --mode=644 -- README.Loki "${pkgdir}/opt/ut/Help/README.Loki"
    install --mode=755 -- bin/Linux/x86/ucc "${pkgdir}/opt/ut/ucc"
    install --mode=755 -- bin/ut "${pkgdir}/opt/ut/ut"
    ln -fs -- /opt/ut/ut "${pkgdir}/usr/bin/ut"

    tar xfC data.tar.gz "${pkgdir}/opt/ut" \
            --exclude=System/UnrealTournament.ini.PATCH
    chmod 644 -- "${pkgdir}/opt/ut/System/OpenGLDrv.int"
    install --mode=644 -D -- "${pkgdir}/opt/ut/System/License.int" \
            "${pkgdir}/usr/share/licenses/${pkgname}/License.int"

    tar xfC UT436-OpenGLDrv-Linux-090602.tar.gz "${pkgdir}/opt/ut"
    tar xfC OpenGL.ini.tar.gz "${pkgdir}/opt/ut"
    tar xfC Credits.tar.gz "${pkgdir}/opt/ut"
    tar xfC NetGamesUSA.com.tar.gz "${pkgdir}/opt/ut"

    # As there is no distinction between GOTY and non-GOTY CDs yet, we just try
    # to patch everything that applies.  Also Loki's patcher is too unreliable.
    cd "${srcdir}"
    echo "Trying to apply Loki's 436 Xdelta patches..."
    _unreal_fail_safe_patcher 436/setup.data/data "${pkgdir}/opt/ut"
    _unreal_fail_safe_patcher 436goty/setup.data/data "${pkgdir}/opt/ut"
#   ./436/setup.data/bin/Linux/x86/loki_patch \
#           ./436/setup.data/patch.dat "${pkgdir}/opt/ut"
#   ./436goty/setup.data/bin/Linux/x86/loki_patch \
#           ./436goty/setup.data/patch.dat "${pkgdir}/opt/ut"

    echo "Applying 451 UTPG patch..."
    tar xfC UTPGPatch451.tar.bz2 451utpg
    _unreal_move_files 451utpg "${pkgdir}/opt/ut" < utpg.list

    # Fix a small bug until next UTPG release.  Thanks for the hint, elsixdiab.
    sed -i '/^LoadClassMismatch/s/\.%s$//' \
            "${pkgdir}/opt/ut/System/Core.int"
}

package() {
    source /usr/lib/makepkg/unreal.sh

    if [ -z "${_cdpath}" ]; then
        _detect_cdpath
    else
        echo "Using ${_cdpath} as install medium."
    fi
    if [ -z "${_cdversion}" ]; then
        _detect_cdversion
    else
        echo "Using \"${_cdversion}\" install method."
    fi
	
	#
    install --directory -- "${srcdir}"/{436,436goty,451utpg} \
            "${pkgdir}/usr/bin" \
            "${pkgdir}"/opt/ut/{Help,Logs,Maps,Music,Sounds,System,Textures} \
            "${pkgdir}"/opt/ut/Web/{images,inc,plaintext/inc}
    install --mode=644 -D -- "${srcdir}/ut.desktop" \
            "${pkgdir}/usr/share/applications/ut.desktop"

    case "${_cdversion}" in
        ("default")
            _build_default
            ;;
        ("anthology")
            _build_anthology
            ;;
        (*)
            echo "Unknown _cdversion: ${_cdversion}" >&2
            ;;
    esac
}
