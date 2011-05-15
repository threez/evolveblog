---
layout: post
title: iStatd Package for Arch Linux (PKGBUILD)
tags: istatd package arch linux pgkbuild
---

I play around with [arch linux](http://www.archlinux.org). It is a very clean and nice system and i like the way it works. One thing I often don't like very much on linux systems, is the creation of custom packages. Package managers like rpm, deb and so on. In my opinion the learning curve is some times a bit hard a they often miss a simple structure.

To not deal with this issues on my mac system i use [homebrew](http://mxcl.github.com/homebrew/). I like the easy way to build custom packages most. This makes the use of fancy software a lot easier when it comes to software deployment. Since homebrew is mac only i have to deal with the *arch* linux provided tools.

And suprise, suprise they aren't that bad. Especially the documentation i found on [Creating Packages](https://wiki.archlinux.org/index.php/Creating_Packages) was very helpful. What it all boils down to is, that one has to write one file with similar content like in *homebrew* [formulars](https://github.com/mxcl/homebrew/wiki/Formula-Cookbook) and you're done. Look at this:

{% highlight bash %}
pkgname=istatd
pkgver=0.5.7
pkgrel=2
pkgdesc="Serving statistics to the iStat iPhone application from Linux, Solaris and FreeBSD."
url="https://github.com/tiwilliam/istatd"
arch=('x86_64' 'i686')
license=('MIT')
depends=('libxml2')
optdepends=()
makedepends=()
conflicts=()
replaces=()
backup=()
install=
source=("https://github.com/downloads/tiwilliam/istatd/${pkgname}-${pkgver}.tar.gz" "patch.diff")
md5sums=('15e717be70a64cf8a8522da8ee5641d3' '8411e2df67e8e61f0f8cf436fdd76e94')

build() {
  cd "${srcdir}/${pkgname}-${pkgver}"
  patch -p0 < ../patch.diff
  ./configure --prefix=/usr
  make
}

package() {
  cd "${srcdir}/${pkgname}-${pkgver}"
  
  make DESTDIR="$pkgdir/" install
  mkdir -p $pkgdir/var/{run,cache}/istat
}
{% endhighlight %}

Patching the istatd server i needed was also very simple. Just include a patch file and use it in your build description. The Files can be found [here](https://gist.github.com/973441). After putting them in a directory one can start the package building by executing the build script:

    makepkg -s

After the build is complete you should get a **istatd-0.5.7-2-i686.pkg.tar.xz** file. This file is the precompiled package that can be passed around and installed easily by using [pacman](https://wiki.archlinux.org/index.php/Pacman):

    pacman -U istatd-0.5.7-2-i686.pkg.tar.xz
    
Uninstalling the software using:

    pacman --remove istatd
    
I highly encourage you to have a look at arch linux. Stay tuned...
