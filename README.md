Introduction
============

Very basic, custom but barely customizable udeb replacing partman
from debian-installer. It creates LVM LVs utilizing the RAID1 mirror feature
of LVM, yet striping swap on both disks. It would have been possible
to use MD-RAID but that would have disabled us from striping swap etc.

It also creates a /boot partition on the first disk. Furthermore it leaves
a partition free at the end to be used as ZFS cache volume later.

There are multiple variables to tweak whilest installing in a preseeded
configuration. These can be found in `debian/partrep.templates`.

It is recommended to take this as template for writing your own shell script
to perform partitioning if you are not happy with partman-auto. You may
try to extend that script to fit your partitioning scheme but that will
probably end in a mess. If you did extend it, the author will be happy to
accept pull requests. ;-)

Usage
=====

First build the `partrep` package like a normal debian package, e.g. using
`debuild -us -uc`. This will not result in a `.deb` but in in an `.udeb`
package without any futher configuration.

To use this in debian-installer you have to build your own image.
This sounds hard but it is a very easy procedure:

* create a container of the desired Debian release and enable all deb
  and deb-src repos. Especially the latter are important. Run `apt-get update`.
* `apt-get -y build-dep debian-installer`
* `apt-get source debian-installer`
* `cd debian-installer*`
* `dpkg-checkbuilddeps`
* `cd build`
* Edit `config/common` and change everything (udeb source as well as the
  target release) to fit your desired release
* `cp /somewhere/partrep_*.udeb localudebs/`
* `echo "partrep" >> pkg-lists/netboot/local`
  `echo "partman-base -" >> pkg-lists/netboot/local`
  Notice there is a minus (`-`) after the `partman-base ` in the last entry!
  This will _exclude_ partman-base from the final image. Check in
  `MANIFEST.udebs` after building for partrep to be included and partman-base
  missing. This will not disable the installer from fetching partman-base, but
  it will not be used since partrep provides the partman-base target.
* `make build_netboot`

After that your netboot release will be found in `dest/netboot/netboot.tar.gz`.
It may be possible to build CD images accordingly but that is not tested yet.

References
==========

* <http://ftp.gnome.org/pub/debian-meetings/2006/debconf6/slides/Debian_installer_workshop-Frans_Pop/paper/>
* <http://blog.loftninjas.org/2007/07/04/complex-lvm-on-an-alternative-install-of-ubuntu-debian-installer/>
* <https://wiki.debian.org/DebianInstaller/Modify/CustomKernel>
* <https://salsa.debian.org/installer-team/debian-installer/blob/master/doc/devel/modules.txt>
* <https://salsa.debian.org/installer-team/partman-base/tree/master/debian>

Just a small hint: You may download and examine udebs easily by spinning up
a container or chroot and replacing the repositories in `/etc/apt/sources.list`
with the following line:
```
deb http://deb.debian.org/debian buster main/debian-installer
```
This will enable you to use the ususal debian package management tools to deal
with udebs of your current release. Even `apt-get install` works -- but will
likely break your system, of course.

