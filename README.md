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
second argument is the Podman image name to be created. Third,
optional argument is the name of the initial user, otherwise defaults
to your account name.

```sh
sudo ./convert_rpi_to_podman ~/Downloads/2023-05-03-raspios-bullseye-arm64-lite.img.xz raspi64
```

Please note that this imports the container to your own Podman
storage. It detects the user you are running sudo with and places the
resulting image there.

The conversion takes minutes. Due to Raspberry Pi OS initialization
oddities, we must run it initially with full system emulation to keep
the file system resize scripts happy. Also, some of the supplied one-time
services depend on `multi-user.target` instead of `basic.target`, so
this conversion must pull the system to multi-user mode before
conversion. **A misleading login screen** appears while converting. No
need to login there. Just relax until it says *Ready*.

## Running the container

After conversion, a static user emulator binary is placed in the
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

It does auto login on startup. If that's not what you want, you may
remove file
`/etc/systemd/system/console-getty.service.d/override.conf` inside the
container.

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

## Author

Written by Joel Lehtonen / Zouppen. Licenced under the terms of MIT license.
