# proxmox-datacenter-manager-arm64

[![Build]][build_url]
[![Version]][release_url]
[![Size]][release_url]

Script for building Proxmox Datacenter Manager **1.x** for ARM64.

## Download pre-built packages

You can find unofficial Debian packages that are created with this script in the [Releases](https://github.com/qemus/proxmox-datacenter-arm64/releases) section.

With the script you can also download or install all packages of the latest release automatically.

**Download and install**

`./build.sh install` or a specific version `./build.sh install=1.1.4`

**Download only**

`./build.sh download` or a specific version `./build.sh download=1.1.4`

For an even easier experience, you can also use the [Datacenter Manager Docker container](https://github.com/dockur/proxmox-datacenter), which is built on top of these same packages.

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

The compilation will take about 10 minutes.  
After that you can find the finished packages in the folder `packages/`.

## Build using Docker

You can build ARM64 `.deb` packages using the provided Dockerfile and docker buildx:

```bash
docker buildx build -o packages --platform linux/arm64 .
```

You can also set build arguments for base image and build.sh options:

```bash
docker buildx build -o packages --build-arg buildoptions="debug" --build-arg baseimage=debian:trixie-slim --platform linux/arm64 .
```

Once the Docker build is completed, packages will be copied from the docker build image to a folder named `packages` in the root folder.

## Build using cross compiler

### Enable multi arch and install build essentials and dependencies

For cross compiling you need to enable multiarch and install the needed build dependencies for the target architecture. The

```bash
dpkg --add-architecture arm64
```

```bash
apt update && apt-get install -y --no-install-recommends \
	build-essential crossbuild-essential-arm64 curl ca-certificates sudo git lintian jq rsync \
	pkg-config pkgconf:arm64 libudev-dev:arm64 libssl-dev:arm64 libapt-pkg-dev:arm64 \
	libclang-dev libpam0g-dev:arm64 libcrypt-dev:arm64 libsystemd-dev:arm64 \
	libacl1-dev:arm64 uuid-dev:arm64 libfuse3-dev:arm64 libldap2-dev:arm64 \
	libzstd-dev:arm64 zlib1g-dev:arm64 nettle-dev:arm64 \
	qemu-user qemu-user-binfmt patchelf
```

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

Install all generated packages:

```bash
sudo apt install ./packages/*.deb
```

## Help section

### Debugging

You can add the debug option to redirect the complete build process output also to a file (`build.log`):

```bash
./build.sh debug
```

## Acknowledgements

Special thanks to [wofferl](https://github.com/wofferl), this project would not exist without his invaluable work.

## Stars 🌟
[![Stargazers](https://raw.githubusercontent.com/star-stats/stars/refs/heads/data/charts/qemus-proxmox-datacenter-arm64.svg)](https://github.com/qemus/proxmox-datacenter-arm64/stargazers)

## Disclaimer ⚖️

*The product names, logos, brands, and other trademarks referred to within this project are the property of their respective trademark holders. This project is not affiliated, sponsored, or endorsed by Proxmox Server Solutions GmbH.*

[build_url]: https://github.com/qemus/proxmox-datacenter-arm64/
[release_url]: https://github.com/qemus/proxmox-datacenter-arm64/releases/

[Build]: https://github.com/qemus/proxmox-datacenter-arm64/actions/workflows/release.yml/badge.svg
[Size]: https://img.shields.io/badge/size-20.3_MB-steelblue?style=flat&color=066da5
[Version]: https://img.shields.io/github/v/tag/qemus/proxmox-datacenter-arm64?label=version&sort=semver&color=066da5
