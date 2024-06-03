# Programming for BBC micro:bit with Rust

## Introduction

This repository provides a template and documentation for setting up a
development environment for (embedded) Rust, specifically targeting the
[BBC micro:bit](https://microbit.org/).

The development environment is designed to run on Ubuntu within a Windows 11
host using WSL 2.

## Hardware

The **BBC micro:bit v2** is used for this project.

- MCU: Nordic nRF52833
  - Core: ARM Cortex-M4 + FPU
  - Clock: 64 MHz
  - Flash: 512 KiB
  - RAM: 128 KiB

Technical details can be found at <https://tech.microbit.org/>.

## Setup the Development Environment

This guide assumes that Ubuntu is installed on Windows 11 using WSL 2, and that
Visual Studio Code is set up with remote access. For more details, refer to the
documentation: <https://code.visualstudio.com/docs/remote/wsl>.

The setup of the development environment consists of multiple steps. Some
prerequisites must be performed on the Windows host system. The actual Rust
instance must be setup on Ubuntu.

### Windows

The host system (Windows) needs to be prepared to forward USB devices to WSL.

This can be achieved with **usbipd** (see
<https://learn.microsoft.com/en-us/windows/wsl/connect-usb>) and can quickly be
set up via *winget*:

```console
winget install --interactive --exact dorssel.usbipd-win
```

The following initial preparation needs to be performed once (for a particular
device):

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

Now, everything is prepared and the device can be attached or detached whenever
access is desired on Windows or Linux:

```console
usbipd attach --wsl --busid 2-5
usbipd detach --busid 2-5
```

### Linux (WSL)

If the USB device is attached to WSL, it should be available on Linux:

```console
$ lsusb | grep -i "NXP ARM mbed"
Bus 001 Device 002: ID 0d28:0204 NXP ARM mbed
```

To be able to use the device without root privileges, a few steps must be
carried out once for the particular device (in this case `001/002`).

The following command can be used to check the privileges. In this case, the
output shows the default privileges (`crw-rw-r--`).

```console
$ ls -ls /dev/bus/usb/001/002
0 crw-rw-r-- 1 root root 189, 1 May 22 17:00 /dev/bus/usb/001/002
```

The access can be extended with a *udev rule*:

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

4. Reconnect the device and check the rules. Please note that the device number
   may have changed:

    ```console
    $ lsusb | grep -i "NXP ARM mbed"
    Bus 001 Device 003: ID 0d28:0204 NXP ARM mbed
    $ ls -l /dev/bus/usb/001/003
    0 crw-rw-rw- 1 root root 189, 2 May 22 17:03 /dev/bus/usb/001/003
    ```

### Prepare Linux

This guide assumes that Rust is already installed and verified on Ubuntu
(<https://rustup.rs/>). I recommend to get familiar with Rust before proceeding
with this guide.

Install GNU Debugger **gdb**:

```console
sudo apt install gdb-multiarch minicom
```

Install Rust embedded debug toolkit **probe-rs**:

```console
curl --proto '=https' --tlsv1.2 -LsSf https://github.com/probe-rs/probe-rs/releases/latest/download/probe-rs-tools-installer.sh | sh
source $HOME/.cargo/env
```

Install Visual Studio Code extension **probe-rs.probe-rs-debugger**.

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
