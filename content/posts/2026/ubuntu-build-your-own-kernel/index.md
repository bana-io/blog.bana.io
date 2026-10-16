---
title: "Ubuntu: Build your own kernel"
description: "Need to build the kernel Ubuntu uses with some custom patches? Well this is for you then epecially if you want to enable HDMI FRL for MTL+ (Intel Meteor Lake and above on older kernels)."
date: 2026-10-15
tags:
  - Ubuntu
  - Linux
  - Kernel
type: blog
---

## Introduction

For whatever reason the (old) Ubuntu wiki pages no longer exists, for instance <https://wiki.ubuntu.com/Kernel/FAQ> no longer works. The old archives are available at <https://github.com/ubuntu/wiki-archives>.

The reason for this post is that I wanted to apply a total of 56 patches, as per the messages to the mailing list <https://sashiko.dev/#/patchset/20261012035806.3361439-1-ankit.k.nautiyal%40intel.com>, to the Linux kernel as these patches were not likely to land in Ubuntu's kernel anytime soon. The patches enable 120Hz refresh rate over HDMI 2.1 with certain Intel chips, without the patches, the refresh rate is limited to 60Hz over HDMI 2.1.

## Prerequisites

Prior to running the below, make sure you have done what is on <https://ubuntu.com/kernel/docs/how-to/source-code/enable-source-repositories/>. If you are on, say, `resolute` after following the above instructions the `/etc/apt/sources.list.d/ubuntu.sources` file should end up like below:

```sh
$ cat /etc/apt/sources.list.d/ubuntu.sources
Types: deb
URIs: http://gb.archive.ubuntu.com/ubuntu/
Suites: resolute resolute-updates resolute-backports
Components: main restricted universe multiverse
Signed-By: /usr/share/keyrings/ubuntu-archive-keyring.gpg

Types: deb
URIs: http://security.ubuntu.com/ubuntu/
Suites: resolute-security
Components: main restricted universe multiverse
Signed-By: /usr/share/keyrings/ubuntu-archive-keyring.gpg

Types: deb deb-src
URIs: http://archive.ubuntu.com/ubuntu
Suites: resolute resolute-updates resolute-backports
Components: main restricted universe multiverse
Signed-By: /usr/share/keyrings/ubuntu-archive-keyring.gpg

```

If you want a one-liner, then it is:

```sh
sudo sed -i 's/Types: deb$/Types: deb deb-src/g' /etc/apt/sources.list.d/debian.sources
sudo apt update
```

Consult <https://ubuntu.com/kernel/docs/how-to/develop-customise/build-kernel/> and <https://github.com/ubuntu/wiki-archives/blob/main/UbuntuWiki/K/Kernel-BuildYourOwnKernel.wiki> if anything goes wrong, that said the steps are:

```sh
sudo apt build-dep linux linux-image-unsigned-$(uname -r)
sudo apt install libncurses-dev gawk flex bison openssl libssl-dev dkms libelf-dev libudev-dev libpci-dev libiberty-dev autoconf llvm
sudo apt install git

sudo apt update && \
    sudo apt build-dep -y linux linux-image-unsigned-$(uname -r) && \
    sudo apt install -y fakeroot llvm libncurses-dev dwarves

sudo apt install -y libunwind8-dev libdwarf-dev \
  libelf-dev libdw-dev systemtap-sdt-dev \
  libssl-dev libslang2-dev binutils-dev \
  libzstd-dev libbabeltrace-dev libiberty-dev \
  libnuma-dev libcap-dev
sudo apt install linux-tools-common linux-tools-generic
sudo apt install **python**-dev-is-python3
sudo install clang-format clang-tidy clang-tools clang clangd libc++-dev libc++1 libc++abi-dev libc++abi1 libclang-dev liblldb-dev libllvm-ocaml-dev libomp-dev libomp5 lld lldb llvm-dev llvm-runtime llvm python3-clang
sudo apt install rustc rust-src bindgen rustfmt rust-clippy

mkdir -pv hdmi21_dev
cd hdmi21_dev

apt source linux-image-unsigned-$(uname -r)
cd linux-7.0.0
wget 'https://blog.bana.io/ubuntu-build-your-own-kernel/files/hdmi21_dev.patch'
patch -p1 < ./hdmi21_dev.patch
```

## Build The Kernel

```sh
chmod a+x debian/scripts/*
chmod a+x debian/scripts/misc/*
fakeroot debian/rules clean
```

Modify `debian.master/changelog` as documented on <https://ubuntu.com/kernel/docs/how-to/develop-customise/build-kernel/#modify-abi-number>, and also you should use the version of `rustc` as supplied by your package manager, essentially for now just remove the `cargo` and `rustup` directories from `PATH`:

```sh
export RUST_LIB_SRC=/usr/src/rustc-$(rustc --version | cut -d' ' -f2)/library
export do_linux_main_modules_depends=false
fakeroot debian/rules clean
fakeroot debian/rules binary-headers binary-generic binary-perarch
fakeroot debian/rules binary
```

## Install The New Kernel

```sh

```

