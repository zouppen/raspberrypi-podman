# Convert Raspberry Pi OS image to a container

Converts Raspberry Pi OS image to a Podman container.

Input can be any 64 bit Raspberry Pi OS. Tested with *bullseye*. The
extracted image may not exceed 4GB (or change the value from the
script. This shouldn't be a problem since you most likely want to use
*lite* variant anyway.

## Requirements

Debian/Ubuntu requirements:

```sh
sudo apt install qemu-user-static qemu-system-arm xz-utils podman
```

## Conversion

First, obtain Raspberry Pi OS (64-bit) Lite from [raspberrypi.com](https://www.raspberrypi.com/software/operating-systems/).

Then run conversion. The first argument is the path to the .xz image,
second argument is the image name. Third, optional argument is the
name of the initial user, otherwise defaults to your account name.

```sh
sudo ./convert_rpi_to_podman ~/Downloads/2023-05-03-raspios-bullseye-arm64-lite.img.xz raspi64
```

Please note that this imports the container to your own Podman
storage. It detects the user you are running sudo with and places the
resulting image there.

The conversion takes minutes. Due to Raspberry Pi OS oddity, some
initial services depend on `multi-user.target` instead of
`basic.target`, so this conversion must pull the system to multi-user
mode before conversion. A misleading login screen appears while
converting. No need to login there. Just relax until it says *Ready*.

## Running the container

After conversion, qemu packages are no longer needed. A static user
emulator binary is placed in the container root filesystem and is
automatically used.

To run a throwaway container:

```sh
podman run --rm=true -it raspi64
```

To create container with your hostname of choice and a shared directory:

```sh
podman create --hostname=vadelma --name=vadelma -v raspi_share:/mnt raspi64
podman start -it vadelma
```

It does auto login on startup. If that's not what you want, you may
remove file
`/etc/systemd/system/console-getty.service.d/override.conf` inside the
container.

