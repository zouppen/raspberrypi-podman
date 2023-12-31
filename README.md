# Convert Raspberry Pi OS image to a container

Converts Raspberry Pi OS image to a Podman container.

Input can be any 64 bit Raspberry Pi OS. Tested with *bullseye*. The
converted image may not exceed 4GB (feel free to enlarge the value in
the script if you need more). This shouldn't be a problem since you
most likely want to use *lite* variant anyway.

## Requirements

Debian/Ubuntu requirements:

```sh
sudo apt install qemu-user-static qemu-system-arm xz-utils podman
```

## Conversion

First, obtain Raspberry Pi OS (64-bit) Lite from [raspberrypi.com](https://www.raspberrypi.com/software/operating-systems/).

Then run conversion. The first argument is the path to the .xz image,
second argument is the Podman image name to be created.

```sh
sudo ./convert_rpi_to_podman ~/Downloads/2023-05-03-raspios-bullseye-arm64-lite.img.xz raspi64
```

Please note that this imports the container to your own Podman
storage. It detects the user you are running sudo with and places the
resulting image there.

The conversion takes minutes. Due to Raspberry Pi OS initialization
oddities, we must run it initially with full system emulation to keep
the file system resize scripts happy.

## Running the container

During conversion, a static user emulator binary is placed in the
container root filesystem and is automatically used. See
[QemuUserEmulation](https://wiki.debian.org/QemuUserEmulation) for
technical details.

To run a throwaway container:

```sh
podman run --rm=true -it raspi64
```

To create a container with your hostname of choice and a shared directory:

```sh
podman create -it --hostname=vadelma --name=vadelma -v /path/to/share:/mnt/share raspi64
```

To start it, just run `podman start -a vadelma`.

On first run new user is prompted. After user creation, auto login
happens. If that's not what you want, you may revert it by running
inside the container:

```
systemctl revert console-getty
rm /etc/console-getty
```

## How about container to SD card conversion?

The containerized system is manipulated carefully so that the systemd
overrides are activated only when the system is running inside
Podman. So, converting container back to SD card image should be
possible.

Just create a smaller VFAT partition for /boot and rest as ext4
partition, copy files accordingly and adjust `/etc/fstab`. It should
work! In case you create a tool, I'm happy to merge it.

## Issues

QEMU user emulation doesn't emulate system calls but pass them to the host, so
don't expect everything to work like in a native system. However,
running on x86_64 syscalls are mostly compatible.

### AppArmor

AppArmor is stupid enough to enforce rules in a different mount namespace,
too. QEMU user emulation requires read access to the binary itself to
be able to emulate it. Reading the binary is prohibited by some
profiles such as `man`.

Follow system journal to see if there are AppArmor errors in the
log. If so, disable (or improve) those rules.

To get man pages working, you can disable the whole rule. On the host
system:

```sh
sudo ln -s /etc/apparmor.d/usr.bin.man /etc/apparmor.d/disable/
```

## Author

Written by Joel Lehtonen / Zouppen. Licenced under the terms of MIT license.
