# Programming for BBC micro:bit with Rust

## Introduction

This repository is a template and documentation for a development environment
for (embedded) Rust. The target platform is a
[BBC micro:bit](https://microbit.org/).

The development environment runs on Ubuntu on Windows 11 host (WSL 2).

## Hardware

This project covers **BBC micro:bit v2**:

- MCU: Nordic nRF52833
  - Core: ARM Cortex-M4
  - Flash: 512 KiB
  - RAM: 128 KiB
  - Clock: 64 MHz

## Setup

- VSC
- Ubuntu WSL 2

The setup of the development environment consists of multiple steps. Some
prerequisites must be performed on the Windows host system. The actual Rust
instance must be setup on Ubuntu.

### Windows

Some steps need to be done to forward USB devices to WSL. **usbipd** is used to
forward USB devices to WSL 2 (see
<https://learn.microsoft.com/en-us/windows/wsl/connect-usb>).

The installation via *winget* is very simple:

```console
winget install --interactive --exact dorssel.usbipd-win
```

Forward a device:

1. Connect the device (BBC micro:bit) to the PC via USB.
2. Open Administrator PowerShell and get a list of  all connected USB devices:

    ```console
    $ usbipd list
    Connected:
    BUSID  VID:PID    DEVICE                                                        STATE
    2-5    0d28:0204  USB Mass Storage Device, USB Serial Device (COM3), USB In...  Not shared
    ```

    In this case, the device has the bus id `2-5`.

3. With the bind command, the device must be prepared for shared use.

    ```console
    usbipd bind --busid 2-5
    ```

4. Now, everything is prepared and the device can be attached or detached:

    ```console
    usbipd attach --wsl --busid 2-5
    usbipd detach --busid 2-5
    ```

### Linux (WSL)

The device should be available under Linux now:

```console
$ lsusb | grep -i "NXP ARM mbed"
Bus 001 Device 002: ID 0d28:0204 NXP ARM mbed
```

Before the device can be used, the access rights for the particular device (in
this case 001/002) should be checked:

```console
$ ls -ls /dev/bus/usb/001/002
0 crw-rw-r-- 1 root root 189, 1 May 22 17:00 /dev/bus/usb/001/002
```

In this case, root privileges are required to access the device.
This can be changed by adding a *udev rule*:

1. Create a new file:

    ```console
    vim /etc/udev/rules.d/99-microbit.rules
    ```

2. Paste the following content:

    ```text
    # CMSIS-DAP for microbit
    SUBSYSTEM=="usb", ATTR{idVendor}=="0d28", ATTR{idProduct}=="0204", MODE:="666"
    ```

3. Reload the udev rules:

    ```console
    sudo udevadm control --reload-rules`
    ```

4. Reconnect the device and check the rules. Please not that the device number
   may has changed:

    ```console
    $ lsusb | grep -i "NXP ARM mbed"
    Bus 001 Device 003: ID 0d28:0204 NXP ARM mbed
    $ ls -l /dev/bus/usb/001/003
    0 crw-rw-rw- 1 root root 189, 2 May 22 17:03 /dev/bus/usb/001/003
    ```

### Prepare Linux

Install Rust.

Install gdb:


```console
sudo apt install gdb-multiarch minicom
```

Install probe-rs:

```console
curl --proto '=https' --tlsv1.2 -LsSf https://github.com/probe-rs/probe-rs/releases/latest/download/probe-rs-tools-installer.sh | sh
source $HOME/.cargo/env
```

## New Project

```console
cargo new firstproject
cd firstproject
rustup target add thumbv7em-none-eabihf
cargo embed --target thumbv7em-none-eabihf
```

<https://crates.io/crates/microbit-v2>

## Debug

```console
gdb-multiarch target/thumbv7em-none-eabihf/debug/rustmicrobit
```

`GDB stub listening at 127.0.0.1:1337`

```console
target remote :1337
```
