# Case Study Building and Running

This section builds and runs the `security_demo` demonstration application described in [Case Study Introduction](case_study_intro.md). All the host machine and target platform requirements described previously in this developer kit documentation are assumed.

## Building the Application

As usual, this assumes that the user is already running a Docker container within the [build environment](build_environment_setup.md), where we can create a directory and clone the code and dependencies.

```text
mkdir /host/security_demo
cd /host/security_demo
```

```bash
repo init -u https://github.com/sel4-cap/dev-kit-microkit-manifest
```

```bash
repo sync
```
From the `/host/security_demo` directory, we execute the following commands:

```text
cd microkit
```

```bash
./init-build.sh -DMICROKIT_APP=security_demo -DPLATFORM=maaxboard
```

## Preparing to Run

A successful build will result in an executable file called `sel4_image` in the `example/<platform>/security_demo/example-build` subdirectory. This file should be made available to the preferred loading mechanism, such as TFTP, as per [Execution on Target Platform](execution_on_target_platform.md).

Running the `security_demo` application requires the following:

- Connect a keyboard to the USB socket[^1] of the MaaXBoard;

[^1]: Note: Currently, only the upper USB port on the Avnet MaaXBoard is active (i.e. the port furthest away from the PCB); the lower USB port does not function. This is a feature of the power domains on the board, not the USB driver.

## Running the Application

The application invokes two instances of the [U-Boot Driver Library](uboot_driver_library.md), so various sets of diagnostic messages are repeated on the CoolTerm display as the application starts. We should not be unduly concerned with some of the individual messages, such as `No ethernet found`, since in this case none of the library instances are configured to use ethernet. There are also some `clk_register: failed ... (parent ...)` messages, which are harmless (a fault in U-Boot's clock driver for the MaaXBoard).

When the application's initialisation has completed, we should see:

```text
run_uboot_command@uboot_wrapper.c:176 --- running command 'fatrm mmc 0:2 transmitter_log.txt' ---
Net:   transmitter_log.txt: doesn't exist
run_uboot_command@uboot_wrapper.c:181 --- command 'fatrm mmc 0:2 transmitter_log.txt' completed with return code 1 ---
```

Note that on subsequent runs, the log file will exist (unless the user has intentionally deleted it), and the output will instead read:

```text
run_uboot_command@uboot_wrapper.c:176 --- running command 'fatrm mmc 0:2 transmitter_log.txt' ---
run_uboot_command@uboot_wrapper.c:181 --- command 'fatrm mmc 0:2 transmitter_log.txt' completed with return code 0 ---
```

In either scenario, this is housekeeping by the application to delete any previous Transmitter logfile from the SD card, before it starts writing new log data. The logfile is named `transmitter_log.txt` and is expected on the third partition of the SD card - see the FAT partition `FILESYS` established during the [Partitioning the SD Card appendix](appendices/partitioning_sd_card.md).

The application is now ready to perform various actions concurrently:

1. If a key is pressed, the plaintext character will be encrypted into a ciphertext character;
2. When the circular buffer is full, the application will append them to the logfile on the SD card.

Periodically, the CoolTerm window will show diagnostic messages concerning the logfile, such as:

```text
run_uboot_command@uboot_wrapper.c:176 --- running command 'fatwrite mmc 0:2 0x55b010 transmitter_log.txt 26 1' ---
38 bytes written in 5 ms (6.8 KiB/s)
run_uboot_command@uboot_wrapper.c:181 --- command 'fatwrite mmc 0:2 0x55b010 transmitter_log.txt 26 1' completed with return code 0 ---
```

The application will continue indefinitely.

If the MaaXBoard is powered off and its SD card removed and transferred to the host machine, the `transmitter_log.txt` can be accessed from the `FILESYS` partition as one would expect.

To reduce the risk of corrupting the SD card, it is advisable to avoid powering off the MaaXBoard during a write operation of the logfile. As this operation lasts in the order of 10 ms, the risk is both remote and avoidable (e.g. power off soon after a `fatwrite` command message).
