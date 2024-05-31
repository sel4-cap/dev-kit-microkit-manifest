# Case Study Design Detail

Following on from the [case study introduction](case_study_intro.md), this section provides further details on the design and functionality of the 'security domain' demonstrator application.

It is expected that the reader is familiar with the [seL4 microkit manual](https://github.com/seL4/microkit/blob/main/docs/manual.md).

## Code Structure

The application is held within the following structure (all within the `microkit` folder), with key folders and files shown:

```text
microkit
│
├───crypto
|   └───crytpo.c
├───keyreader
│   └───keyreader.c
├───transmitter
│   └───transmitter.c
│      
├───include
│   └───plat
│       └───<platform>
│            ├───mmc_platform_devices.h
│            └───usb_platform_devices.h
│
│
├───CMakeLists.txt
│
├───example
│   └───<platform>
│       └───security_demo
│           └───security_demo.system
```

- `CMakeLists.txt`: Application build file.
- `security_demo.system`: Microkit file for the project, declaring the application's protection domains including declaration of all protection domain channels.
- `include/plat/<mmc|usb>_platform_devices.h`: Declaration of (platform specific) microkit attributes to grant a protection domain with capabilities access to a hardware device.

## Concurrency Model

From a concurrency perspective, the security demonstrator can be summarised as threads of execution on the 'high-side' managing plaintext data running asynchronously to threads of execution on the 'low-side' managing ciphertext data.

Threads of execution on the high-side perform the following functions:

1. Reading plaintext keypresses from the USB keyboard;
2. Encrypting those keypresses from plaintext to ciphertext; and
3. Placing ciphertext keypresses into a shared circular buffer between the high-side and low-side.

Asynchronously, threads of execution on the low-side perform the following functions:

1. Reading ciphertext keypresses from a shared circular buffer between the high-side and low-side;
2. Periodically writing received ciphertext to a log file.

All data is passed from the high-side to the low-side through a shared circular buffer stored within a dataport. It is this circular buffer between the high-side and low-side that permits:

1. Data transfer from the high-side to the low-side; and
2. Threads of execution on the high-side and threads of execution on the low-side to run asynchronously.

## Example of Protection Domain Communication

The circular buffer shared between the Crypto protection domain (high-side) and the Transmitter protection domain (low-side) is used in this section as a detailed worked example of microkit data flow and control flow.

The buffer has been deliberately designed for the purpose of this worked example to use two types of protection domain interface listed in the [seL4 microkit manual](https://github.com/sel4-cap/microkit-old/blob/main/docs/manual.md), i.e. Memory region and Notification.

### Memory Region

At its core the circular buffer is a simple character array with *head* and *tail* holding indexes associated with the start and end of the used portion of the array.

- Function implementations for the circular buffer and included in `src/circular_buffer.c`.
- A memory region is a contiguous range of physical memory and the memory region for the circular buffer is mapped onto both the Crypto and Transmitter protection domains. A virtual address, caching attributes and permissions (read, write and execute) are given to each protection domain.

This results in an instance of the circular buffer type being made available in an area of memory shared by both the Crypto and Transmitter protection domains.

### Notification

A mechanism for the Transmitter protection domain to determine whether the circular buffer contains any data.

A Notification interface is used to allow the Crypto protection domain to notify Transmitter when the circular buffer is full. This allows the Transmitter protection domain to wait or poll for such a notification before accessing the circular buffer.

- Channels are set up between the Crypto and Transmitter protection domains in the applications system file which allow the protection domains to notify each other when there is data to read in a shared buffer.

This results in the creation of a notification that can be emitted from the Crypto protection domain and can be either waited on, or polled, by the Transmitter protection domain.

### Summary

Putting the two interface types (Memory region and Notification) together results in:

- A shared circular buffer;
- A notification mechanism to allow the producer to notify the consumer when new data is available.

This thereby allows asynchronous data transfer and buffering between protection domains such that the producer and consumer can work concurrently.

The example source code for the producer protection domain demonstrating use of these communication mechanisms can be found in `crypto/crypto.c`. The source code for the consumer protection domain can be found in `transmitter/transmitter.c`.
