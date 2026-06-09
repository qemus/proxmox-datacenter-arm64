# proxmox-datacenter-manager-arm64

Script for building Proxmox Datacenter Manager **1.x** for **Debian/Trixie**

At least 4 GB are required for compiling. On devices with low memory, SWAP must be used (see help section).

## Download pre-built packages

You can find unofficial debian packages for **Trixie** that are created with the build.sh script and github actions at https://github.com/qemus/proxmox-datacenter-arm64/releases.

With the script you can also download all files of the latest release at once

**Download and install**

`./build.sh install` or a specific version `./build.sh install=1.1.4-1`

**Download only**

`./build.sh download` or a specific version `./build.sh download=1.1.4-1`

## Build manually

### Install build essentials and dependencies

```bash
apt-get install -y --no-install-recommends \
	build-essential curl ca-certificates sudo git lintian fakeroot jq rsync \
	pkg-config libudev-dev libssl-dev libapt-pkg-dev libclang-dev \
	libpam0g-dev libcrypt-dev libacl1-dev libsystemd-dev \
	libfuse3-dev libldap2-dev libzstd-dev \
	zlib1g-dev nettle-dev uuid-dev
```

### Install `rustup`

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs -sSf | sh -s
source ~/.cargo/env
```

### Start build script

```bash
./build.sh
```

The compilation can take several hours.  
After that you can find the finished packages in the folder `packages/`.

## Build using docker

You can build arm64 `.deb` packages using the provided Dockerfile and docker buildx:

```bash
docker buildx build -o packages --platform linux/arm64 .
```

You can also set build arguments for base image and build.sh options:

```bash
docker buildx build -o packages --build-arg buildoptions="debug" --build-arg baseimage=debian:trixie-slim --platform linux/arm64 .
```

Once the docker build is completed, packages will be copied from the docker build image to a folder named `packages` in the root folder.

## Build using cross compiler

### Enable multi arch and install build essentials and dependencies

For cross compiling you need to enable multiarch and install the needed build dependencies for the target architecture. The docs build runs arm64 helper binaries during the build, so `qemu-user` and `qemu-user-binfmt` are needed.

```bash
dpkg --add-architecture arm64
```

```bash
apt update && apt-get install -y --no-install-recommends \
	build-essential crossbuild-essential-arm64 curl ca-certificates sudo git lintian jq rsync \
	pkg-config pkgconf:arm64 libudev-dev:arm64 libssl-dev:arm64 libapt-pkg-dev:arm64 apt:amd64 \
	libclang-dev libpam0g-dev:arm64 libcrypt-dev:arm64 libsystemd-dev:arm64 \
	libacl1-dev:arm64 uuid-dev:arm64 libfuse3-dev:arm64 libldap2-dev:arm64 \
	libzstd-dev:arm64 zlib1g-dev:arm64 nettle-dev:arm64 \
	qemu-user qemu-user-binfmt patchelf
```

`apt:amd64` is necessary because `libapt-pkg-dev:arm64` would break the dependencies without it.

### Install `rustup` and add target arch

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs -sSf | sh -s
source ~/.cargo/env
rustup target add aarch64-unknown-linux-gnu
```

### Start build script

```bash
./build.sh cross
```

## Install packages

```bash
sudo apt install \
	./libjs-extjs_*_all.deb \
	./libproxmox-acme-plugins_*_all.deb \
    ./proxmox-widget-toolkit_*_all.deb \
	./proxmox-datacenter-manager_*_arm64.deb
```

## Help section

### Debugging

You can add the debug option to redirect the complete build process output also to a file (`build.log`):

```bash
./build.sh debug
```

### Create SWAP

At least 4 GB swap is recommended on low memory systems like Raspberry Pi.

Source: https://askubuntu.com/questions/178712/how-to-increase-swap-space/1263160#1263160

Check swap memory:

```bash
swapon --show
free -h
```

Change swapsize on systems with fstab enabled swap:

```bash
sudo swapoff /var/swap
sudo fallocate -l 4G /var/swap
sudo mkswap /var/swap
sudo swapon /var/swap
```

Change swapsize on systems with dphys-swapfile service:

```bash
sudo sed -i "s#.*CONF_\(SWAPSIZE\|MAXSWAP\)=.*#CONF_\1=4096#" /etc/dphys-swapfile
sudo service dphys-swapfile restart
```

## Acknowledgements

Special thanks to [wofferl](https://github.com/wofferl), this project would not exist without his invaluable work.
