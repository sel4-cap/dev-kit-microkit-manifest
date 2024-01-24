# Building Applications

For consistency with the methods employed by the [seL4 Foundation](https://sel4.systems), a [CMake](https://cmake.org) based build system is used. Details on the seL4 build system are documented [here](https://docs.sel4.systems/projects/buildsystem/).

As a worked example throughout this section the [seL4Test](https://docs.sel4.systems/projects/sel4test) project will be used. seL4Test is a test suite for seL4 developed and maintained by the seL4 Foundation.

This section of the document assumes the [build environment setup](build_environment_setup.md) has been completed. All commands provided within this section are to be executed within the build environment; i.e. please ensure that you have followed the instructions in the [build environment setup's usage](build_environment_setup.md#usage) section and that you execute the following commands from inside the Docker container.

## Context

The seL4Test example that is built in this section is a result of an seL4 port for the Avnet MaaXBoard that we created for this developer kit. Whilst our seL4 port is not documented here in detail, there is some guidance on this in the [Guide to Porting seL4](appendices/guide_to_porting_sel4.md) appendix, which includes links to the Git commits that added the MaaXBoard port to the main seL4 repositories.

## Getting the Code

Within the build environment a directory named `seL4Test` will be created and all of the code and dependencies (e.g. the seL4 kernel, libraries and required tools) will be cloned.

Create a directory to hold the code:

```text
mkdir /host/seL4Test
```

```text
cd /host/seL4Test
```

Clone the code and dependencies:

```bash
repo init -u https://github.com/sel4-cap/seL4-dev-kit-microkit-manifest.git
```

```bash
repo sync
```

## Building

Run the init-build.sh in the microkit directory with the example you want to build as one of the arguments 

```bash
# Enter microkit directory
cd /host/seL4Test/microkit
# build application
../init-build.sh "<example you want to build>"
```

On completion of the compilation, the resulting executable is available at `/host/seL4Test/microkit/example/maaxboard/<example built>\example-build` on the build environment, or available at `/<host_directory>/seL4Test/microkit/example/maaxboard/<example built>\example-build` on the host machine, where `/<host_directory>` was the directory on the host mapped to the build environment.

## Appendices

- [Guide to Porting seL4](appendices/guide_to_porting_sel4.md)
