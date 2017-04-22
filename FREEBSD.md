Shairport Sync on FreeBSD
----
Shairport Sync now runs "natively" on FreeBSD using the \*BSD-specific `sndio` back end.

This is an initial note about installing Shairport Sync on FreeBSD.

The build instructions here install back ends for `sndio` (originally developed for OpenBSD) and ALSA. ALSA is, or course, the Advanced Linux Sound Architecture, so it is not "native" to FreeBSD, but has been ported to some architectures under FreeBSD.

General
----
This build was done on a default build of `FreeBSD 11.0-RELEASE-p9`.

First, update everything:
```
# freebsd-update fetch
# freebsd-update install
```
Next, install the `pkg` package manager and update its lists:

```
# pkg
# pkg update
```

Subsystems
----
Install the Avahi subsystem. FYI, `avahi-app` is chosen because it doesn’t require X11. `nss_mdns` is included to allow FreeBSD to resolve mDNS-originated addresses – it's not actually needed by Shairport Sync. Thanks to [reidransom](https://gist.github.com/reidransom/6033227) for this.

```
# pkg install avahi-app nss_mdns
```
Add these lines to `/etc/rc.conf`:
```
dbus_enable="YES"
avahi_daemon_enable="YES"
```
Next, change the `hosts:` line in `/etc/nsswitch.conf` to
```
hosts: files dns mdns
```
Reboot for these changes to take effect.

Building
----

Install the packages that are needed for Shairport Sync to be downloaded and build successfully:
```
# pkg install git autotools pkgconf popt libconfig openssl sndio alsa-utils
```
Omit `alsa-utils` if you're not using ALSA. Likewsie, omit `sndio` if you don't intend to use the `sndio` subsystem.

Now, download Shairport Sync from GitHub and check out the `development` branch.
```
$ git clone https://github.com/mikebrady/shairport-sync.git
$ cd shairport-sync
$ git checkout development
```
Next, configure the build and compile it:

```
$ autoreconf -i -f
./configure  --with-avahi --with-ssl=openssl --with-alsa --with-sndio --with-os=freebsd --with-freebsd-service
$ make
```
Omit `--with-alsa` if you don't want to include the ALSA back end. Omit the `--with-sndio` if you don't want the `sndio` back end. Omit the `--with-freebsd-service` if you don't want to install a FreeBSD startup script, runtime folder and user and group -- see below for more details.

Installation
----

Enter the superuser mode and do a make install.

```
$ su
# make install
```

This will install the `shairport-sync` program along with a sample configuration file at `/usr/local/etc/shairport-sync.conf` and a service startup script to run Shairport Sync as a daemon. In addition, it will define a `shairport-sync` user and group and will create a folder at `/var/run/shairport-sync` to be owned by the user `shairport-sync`. This will be used to hold the daemon's PID file.

Finally, edit `/usr/local/etc/shairport-sync.conf` to customise your installation, e.g. service name, etc. To make the `shairport-sync` daemon load at startup, add the following line to `/etc/rc.conf`:

```
shairport_sync_enable="YES"
```
You can launch the service as superuser, or simply reboot the machine.

Using the `sndio` backend
----

The `sndio` back end does not have a hardware volume control facility. You should set the volume to maximum before use, using, for example, the `mixer` command described below.

Setting Overall  Volume
----
The `mixer` command can be used for setting the output device's volume settings. You may hae to experiment to figure out which settings are appropriate.

```
$ mixer vol 100 # sets overall volume
```
If you've installed `alsa-utils`, then `alsamixer` and friends will also be available.