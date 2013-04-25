# Contributor: fnord0 <fnord0 AT riseup DOT net>

pkgname=acpi_call-git
pkgver=20121230
pkgrel=2
pkgdesc="kernel module that enables calls to ACPI methods through /proc/acpi/call. e.g. to turn off discrete graphics card in a dual graphics environment (like NVIDIA Optimus)"
arch=('i686' 'x86_64')
url=("http://github.com/mkottman/acpi_call")
license=('GPL')
provides=('acpi_call')
makedepends=('git')
optdepends=('linux-headers: needed if using archlinux default kernel'
	    'linux-lts-headers: needed if using the archlinux lts kernel')
install=acpi_call.install
_gitroot=("https://github.com/mkottman/acpi_call.git")
_gitname=("acpi_call")

if [ "$CARCH" = "x86_64" ]; then
  source=('acpi_call-x86_64.patch')
  sha1sums=('2cfb22deca7687b87d3cbd06dbd1d59e73210f62')
fi

build() {
  warning "Please make sure linux kernel headers are built/installed for the kernel acpi_call will be used with ::"
  warning "example #1: 'pacman -S linux-headers'"
  warning "example #2: 'pacman -S linux-lts-headers'"
  cd ${srcdir}

 ## Git checkout
  if [ -d ${srcdir}/${_gitname} ] ; then
    msg "Git checkout:  Updating existing tree"
    cd ${_gitname} && git pull origin
    msg "Git checkout:  Tree has been updated"
  else
    msg "Git checkout:  Retrieving sources"
    git clone ${_gitroot}
  fi
  msg "Checkout completed"

 ## Create an 'acpi_call-build' Directory
  rm -rf ${srcdir}/${_gitname}-build
  cp -r ${srcdir}/${_gitname} ${srcdir}/${_gitname}-build
  cd ${srcdir}/${_gitname}-build

  if [ -d /usr/lib/modules ] ; then
     sed -i 's|/lib/modules/|/usr/lib/modules/|g' ./Makefile || return 1
  fi

 ## x86_64 Patch
  if [ "$CARCH" = "x86_64" ]; then
      msg "Patching for x86_64 systems"
      patch -p1 -i ${srcdir}/acpi_call-x86_64.patch
  fi  

 ## Build
  make
}
package() {
  cd ${srcdir}/${_gitname}-build
  install -d ${pkgdir}/usr/share/${_gitname} || return 1
  install -d ${pkgdir}/usr/bin || return 1
  install -Dm755  ${srcdir}/${_gitname}-build/examples/asus1215n.sh \
    ${pkgdir}/usr/share/${_gitname} || return 1
  install -Dm755 ${srcdir}/${_gitname}-build/examples/dellL702X.sh \
    ${pkgdir}/usr/share/${_gitname} || return 1
  install -Dm755  ${srcdir}/${_gitname}-build/examples/m11xr2.sh \
    ${pkgdir}/usr/share/${_gitname} || return 1
  install -Dm755  ${srcdir}/${_gitname}-build/examples/turn_off_gpu.sh \
    ${pkgdir}/usr/share/${_gitname} || return 1
  ln -s /usr/share/${_gitname}/turn_off_gpu.sh \
    ${pkgdir}/usr/bin/turn_off_gpu.sh || return 1
  install -Dm755  ${srcdir}/${_gitname}-build/support/query_dsdt.pl \
    ${pkgdir}/usr/share/${_gitname} || return 1
  cp -R support/windump_hack \
    ${pkgdir}/usr/share/${_gitname}/
  install -Dm644 README.md \
    ${pkgdir}/usr/share/${_gitname}/README.md

  warning "The following kernel module build procedure *will fail* if your kernel headers are not built/installed!"

 ## the following code was reused from the 'bbswitch-git' PKGBUILD
  _PACKAGES=`pacman -Qsq linux`
  _KERNELS=`pacman -Ql $_packages | grep /modules.alias.bin | sed 's/.*\/lib\/modules\/\(.*\)\/modules.alias.bin/\1/g'`

   # Find all extramodules directories
  _EXTRAMODULES=`find /usr/lib/modules -name version | sed 's|\/usr\/lib\/modules\/||; s|\/version||'`

  # Loop through all detected kernels
  for _kernver in $_KERNELS; do
    msg2 "Building module for $_kernver..."

    # Loop through all detected extramodules directories
    for _moduledirs in $_EXTRAMODULES; do
      # Check which extramodules directory corresponds with the built module
      if [ `cat "/usr/lib/modules/${_moduledirs}/version"` = $_kernver ]; then
        mkdir -p "${pkgdir}/usr/lib/modules/${_moduledirs}/"
        install -D -m644 acpi_call.ko "${pkgdir}/usr/lib/modules/${_moduledirs}/"
        gzip "${pkgdir}/usr/lib/modules/${_moduledirs}/acpi_call.ko"
      fi
    done
  done
}
