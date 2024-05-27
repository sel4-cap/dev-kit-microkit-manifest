# Case Study Introduction

This section works through an example to show some of our device drivers being used within a larger microkit application. It is nominally labelled as a 'security domain' demonstrator (named `security_demo`) but this is mainly in the context of being a keyboard-based encryption device that was inspired by an Enigma machine! It is intentionally simple and its main purpose is to show data separation and to provide worked examples of protection domain communication using different seL4 mechanisms.

Extra functionality within the demonstration application is deliberately kept to a minimum to leave the source code less cluttered, so that the developer can readily see the 'more interesting' protection domain and device interactions without too much extra, feature-supporting code. In line with the rest of the developer kit, the demonstration application is intended to be more of a springboard for a developer kit user than a polished product for an end user.

As with earlier sections of this developer kit documentation, we continue to defer to the seL4 Foundation's [documentation of microkit](https://docs.sel4.systems/projects/microkit/) as the primary source of understanding of microkit, but this section will cover aspects of the use of microkit where appropriate.

## Basic Description

An operator types a plaintext message using a USB-connected keyboard. The application encrypts the message and records the ciphertext messages in a logfile on the SD card of the device.

## Architecture Overview

The architecture of the demonstrator is shown below.

![Demonstrator architecture](figures/encrypter_arch.png)

Arrow directions show an abstracted view of data flow. Arrow labels refer to microkit connector types (some concerned with data flow, some with control flow), which are elaborated in the key. More details about microkit connector types may be found in the [microkit manual](https://github.com/seL4/microkit/blob/main/docs/manual.md), but the fundamental types are Protected procedure and Notification (see examples such as `Protected procedure` in the key).

As the KeyReader and Crypto protection domains handle plaintext and cryptographic data (e.g. keys), they are considered as 'high-side' in terms of security and must be kept separate from the downstream 'low-side' protection domains that handle ciphertext. It is not the role of this developer kit to re-justify the credentials of seL4 (the [seL4 whitepaper](https://sel4.systems/About/seL4-whitepaper.pdf) is a good starting point), but suffice to say that seL4's capability-based access controls guarantee protection and separation between all protection domains, regardless of the notional high and low sides that we have overlaid, only allowing interactions between protection domains where explicitly established via the seL4 connector types.

The following paragraphs briefly describe the data flow, from left to right, highlighting the different seL4 mechanisms used for protection domain communication.

### Protection domains and Connector Types

Plaintext characters are typed on a keyboard and read by the KeyReader protection domain. These characters are then 'encrypted' by the Crypto protection domain to transform them into ciphertext. Protected procedure is an appropriate connector type for the character-by-character data flow between KeyReader and Crypto (labelled as PP on the diagram). Since this application is more concerned with demonstrating seL4 concepts than crypto-algorithms, the Enigma machine's rotors and plugboard are replaced with a simple [ROT13](https://en.wikipedia.org/wiki/ROT13) algorithm!

The encrypted characters are transferred to the Transmitter PD via a shared circular buffer, where Crypto writes to the head of the buffer and Transmitter reads from its tail. The buffer is implemented as a shared memory region.

When the buffer is full Crypto notifies Transmitter that the buffer is full and the data needs to be read via the notification mechanism. Transmitter acts upon notifications by reading all available characters until the buffer is empty.

### Device Drivers

As can be seen from the architecture diagram, three hardware devices are involved in the operation of the application.

1. The KeyReader protection domain requires access to the USB device to allow for plaintext characters to be input from a USB keyboard.

2. The Transmitter protection domain requires access to the SD/MMC device to allow for the ciphertext message to be output to a log file.

Device drivers for the required hardware access are supplied by the [U-Boot Driver Library](uboot_driver_library.md) previously introduced by this development kit.

Two separate instances of the library are used by the application, one per protection domain with a need for hardware device access. The capabilities of each protection domain, and their associated library instances, are configured such that each protection domain is only capable of accessing the minimum set of hardware devices required to perform the desired function.

For example, the Transmitter protection domain only has a need to access the SD/MMC device to write the ciphertext log file. As such, the capabilities of the Transmitter protection domain permit it to access the memory-mapped interface of the SD/MMC device; however no such capabilities are provided for the USB device. Any attempt by the Transmitter protection domain to access the memory-mapped interface of the USB device (e.g. in an attempt to read the plaintext keypresses) would therefore be prevented by seL4.
