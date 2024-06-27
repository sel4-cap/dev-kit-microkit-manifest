# Build Environment Setup

The build environment is the environment within which all binaries for the target environment are built; i.e. the environment that provides the required compilation toolchain.

To simplify the requirements on the host environment and to enable rapid setup, the build environment is provided as a pre-built Docker image.

## Environment Details

The build environment is a Debian Linux system pre-installed with all of the required development tools. The primary tools supplied are:

- [__Git__](https://git-scm.com) version control system.
- [__Repo__](https://gerrit.googlesource.com/git-repo/+/refs/heads/master/README.md) Git repository management tool.
- [__aarch64-linux-gnu__](https://gcc.gnu.org) cross compiler toolchain targeting the ARM AArch64 instruction set.
- [__arm-linux-gnueabi__](https://gcc.gnu.org) cross compiler toolchain targeting the ARM AArch32 instruction set.
- [__Make__](https://www.gnu.org/software/make/), [__CMake__](https://cmake.org), and [__Ninja__](https://ninja-build.org) build automation tools.

## Installation

[Host machine setup](host_machine_setup.md) must be completed, specifically installation of Docker, and the Docker tools must be active prior to installation of the build environment.

Installation of the build environment comprises the download of a pre-built Docker image into the Docker tools. This can be performed from the command line on the host machine as follows:

```bash
git clone "git@github.com:sel4-cap/maaxboard-sel4-dev-env.git"
```

## Usage

When using the build environment, a directory from the host machine must be mapped to the `/host` directory within the build environment; this is the directory within which all work should be performed.

__WARNING__: Any changes made outside the build environment's `/host` directory will be lost when the build environment is exited.

For the purposes of the worked examples in this section the `/home/<user-name>/dev-kit-test/` directory on the host machine is mapped to the build environment's `/host` directory; please note that the directory must be supplied as an absolute path and not as a relative path, and within a Windows environment it needs to include a drive letter (such as `C:/home/<user-name>/dev-kit-test/`). It is expected that the user will choose a different directory and modify the commands as required.

The following command executed from the microkit-maaxboard-dockerfiles folder of the host machine will start the build environment:

```bash
make user IMAGE=sdk HOST_DIR=/home/<user-name>/dev-kit-test/ HOME_DIR=~/
```

On build environment startup the user will be placed into an interactive shell within the `/host` directory.

How the build environment can be used to compile an seL4 binary for the MaaXBoard is documented in the [Building Applications](building_applications.md) section.
