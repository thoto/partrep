Introduction
============

Very basic, custom but barely customizable udeb replacing partman
from debian-installer. It creates LVM LVs utilizing the RAID1 mirror feature
of LVM, yet striping swap on both disks. It would have been possible
to use MD-RAID but that would have disabled us from striping swap etc.

It also creates an /boot partition on the first disk. Furthermore it leaves
an partition free at the end to be used as ZFS cache volume later.

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

To use this in debian-installer image you have to build your own images.
This sounds hard but is a very easy procedure:

* create a container of the desired debian release and enable all deb
  and deb-src repos. Especially the latter are important. Run `apt-get update`.
* `apt-get -y build-dep debian-installer`
* `mkdir di ; apt-get source debian-installer`
* `cd debian-installer*`
* `dpkg-checkbuilddeps`
* `cd build`
* Edit `config/common` and change everything (udebs as well as the target)
  to fit your desired release.
* `cp /somewhere/partrep_*.udeb localudebs/`
* `echo "partrep" >> pkg-lists/netboot/local`
  `echo "partman-base -" >> pkg-lists/netboot/local`
  Notice there is a minus (`-`) after the `partman-base ` in the last entry!
* `make build_netboot`

After that your netboot release will be found in `dest/netboot/netboot.tar.gz`.
It may be possible to build CD images accordingly but that is not tested yet.
