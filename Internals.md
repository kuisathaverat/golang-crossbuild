# Golang Cross build Docker images

This repository contains Dockerfiles for cross building Go binaries for various platforms.
The aim is to provide a simple way to build Go binaries for multiple platforms without having to install and configure cross compilers on your host machine.
To do that the project provide a set of Docker images that can be used to build Go binaries for the following platforms:

* linux/amd64
* linux/arm
* linux/armel
* linux/armhf
* linux/arm64
* linux/mips
* linux/mipsle
* linux/mips64
* linux/ppc64
* linux/ppc64le
* linux/s390x
* windows/amd64
* darwin/amd64
* darwin/arm64

The Docker images are based on Debian and the cross compilers are installed using the crossbuild-essential package.
Each architecture have its own folder with the files needed to build the Docker image for that architecture.
Each architecture has its own Dockerfile to builld a Docker image for that architecture.
Each architecture Dockerfile install the crossbuild-essential package for that architecture and the libraries needed to build our binaries.
These Dockerfiles are generated files from `Dockerfile.tmpl` template file that is in the architectures folder.
This template is processed using a Makefile that is in the architectures folder.
Each architecture folder has a `roofs` folder that contains the files that will be copied to the Docker image in the root folder.
In `rootfs` we have the `compolers.yml` file that contains the list of compilers that will be installed in the Docker image.
Each Docker image has a basic compilation test that is executed when the image is built. This test uses `rootfs/helloworld.c` file to compile a simple C program and verify the architecture of the result binary.
The compiler used to build the binaries is LLVM.
Some of the Docker images are build for the amd64 and arm64 architectures, this allow to run the Docker images in linux/amd64, linux/arm64, darwin/amd64, and darwin/arm64. This is done using the `.ci/scripts/buildx.sh` command.

The Docker images depends on each other, so there is an order to build them.
The are two Docker images that are parent of all the other images, the `fpm` and the `go/llvm-apple`.
Anytime a new Devian version is released, the `fpm` and `go/llvm-apple` image needs to be updated to use the new Debian version.

The Docker images are tagger using the following format:

* `docker.elastic.co/beats-dev/golang-crossbuild:<go-version>-<arch>-<debian-version>`

For the latest version of the images based on the latest Debian and Go versions, the following tags are also used:

* `docker.elastic.co/beats-dev/golang-crossbuild:<go-version>-<arch>`

## Makefiles

There are several Makefiles in the profect across the different folders.
The Makefile in the root folder is used to build the Docker images for the different architectures,
it triggers the build of all dockerimages for all architectures and Devian versions supported.

`go/Makefile.common`
`go/Makefile.debian7`
`go/Makefile.debian8`
`go/Makefile.debian9`
`go/Makefile.debian10`
`go/Makefile.debian11`

## fpm Docker image

This Docker image install the [fpm](https://github.com/jordansissel/fpm) tool that is used to build packages.

## go/llvm-apple Docker image

The LLVM compiler present in Debian does not support arm64e architecture, so we need to build our own LLVM compiler to support this architecture.
The llvm-apple Docker image is based on [Apple LLVM fork](https://github.com/apple/llvm-project) and [osxcross](https://codeload.github.com/tpoechtrager/osxcross). The image build the LLVM compiler and configure the image to be able to cross compile for MacOSX.

LLVM need a SDK for macOS, the MacOSX-SDK used in the Docker image must be genarated from a MacOSX machine, using the instructions at [Packaging the SDK on macOS](https://github.com/tpoechtrager/osxcross#packaging-the-sdk) and uploaded to `https://storage.googleapis.com/obs-ci-cache/beats/MacOSX<macOS-version>.sdk.tar.xz` (e.g. https://storage.googleapis.com/obs-ci-cache/beats/MacOSX11.3.sdk.tar.xz)

## go/base Docker image

## go/base-arm Docker image
