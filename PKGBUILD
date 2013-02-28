# Maintainer: Daniel Hillenbrand <codeworkx@bbqlinux.org>

pkgname=('heimdall')
pkgver=1.4rc2
pkgrel=1
pkgdesc="Heimdall is a cross-platform open-source utility to flash firmware (aka ROMs) onto Samsung Galaxy S devices."
arch=('i686' 'x86_64')
url="http://www.glassechidna.com.au/products/heimdall/"
license=('MIT')
depends=('libusb' 'qt4')
optdepends=('android-udev: Udev rules to connect Android devices to your linux box')
makedepends=('gcc' 'git' 'qt4')

_gitrepo="git://github.com/Benjamin-Dobell/Heimdall.git"
_gittag="v1.12.0"
_gitbranch="master"
_gitname="heimdall"

build() {
  cd "$srcdir"
  msg "Connecting to GIT server..."

  if [ -d $_gitname ] ; then
    cd $_gitname && git pull origin
    msg "The local files are updated."
  else
    git clone $_gitrepo $_gitname
    cd "$_gitname"
    git checkout "$_gitbranch"
    cd "$srcdir"
  fi

  msg "GIT checkout done or server timeout"
  
  rm -rf "$srcdir/$_gitname-build"
  git clone "$srcdir/$_gitname" "$srcdir/$_gitname-build"
  cd "$srcdir/$_gitname-build"

  # Build libpit which is needed for compiling heimdall
  msg "Building libpit..."
  cd libpit/
  ./configure --prefix=/usr
  # Default makefile removes libpit.1.4.a which is needed by frontend
  sed -i '/rm -f libpit-1.4.a/d' Makefile
  make

  # Build heimdall command line tool
  msg "Building heimdall..."
  cd ../heimdall/
  
  ./configure --prefix=/usr
  make

  # Build heimdall GUI front end
  msg "Building heimdall frontend..."
  cd ../heimdall-frontend/

  env OUTPUTDIR="/usr/bin" qmake4 heimdall-frontend.pro
  make
}

package() {
  cd "$srcdir/$_gitname-build"

  # Install heimdall command line tool
  cd heimdall/

  # Prevent make install from trying to reload udev
  # We'll do this the Arch way at package install time
  mv Makefile Makefile.orig
  sed -e 's/sudo service udev restart/echo sudo service udev restart/' <Makefile.orig >Makefile

  make DESTDIR="${pkgdir}" install
  rm -rf "${pkgdir}/lib/"
  install -m644 -D LICENSE "${pkgdir}/usr/share/licenses/${pkgname}/LICENSE"

  # Install heimdall GUI front end
  cd ../heimdall-frontend
  # hack to place heimdall-frontend in /usr/bin
  sed -i 's|local\/||g' Makefile
  make INSTALL_ROOT="${pkgdir}/" install
  install -m644 -D "${srcdir}/heimdall.desktop" "${pkgdir}/usr/share/applications/heimdall.desktop"
}
