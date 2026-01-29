# Nordic SoftDevice Bluetooth Controller with Support for LE ISO Timesync

This fork of the Zephyr HCI UART example adds a custom HCI Command to allow for timesync between the Bluetooth Host 
and the Bluetooth Controller. When the HCI LE Read ISO Clock command is received, the Controller raise a GPIO for
a few microseconds and returns its current Bluetooth LE ISO Clock in the Command Complete Event.

We raise the GPIO instead of just toggling as e.g. STM32F4xx only supports timer capture on a specific edge.

It has been tested on all variants documented below.

## Requirements
- nRF Connect SDK v3.2.1 or newer

## HCI LE Read ISO Clock Command
- OGF: 0x3f, OCF: 0x200
- Parameters: Flags (1 Octet) unused
- Response: HCI Command Complete Event with status and 4 bytes timestamp in microseconds


## HCI H4 Baudrate Config

For USB CDC variants, data is sent and received via USB without the data ever beeing sent over a physical UART lines.

For others, especially on nRF54L15 which does not support USB Device role, the UART uses 1 mbps = 1000000 baud, which differs from the quasi standard 115200 of most Bluetooth Controllers.

Please set .baudrate_init = 1000000 in the trasport config struct of btstack/port/posix-h4/main.c


## nRF58233 Development Kit

The first  Virtual UART (UART1, ...) is Zephyr UART 0
The second Virtual UART (UART2, ...) is Zephyr UART 1

### Pinout

Signal direction as seen from the nRF52833.

| PIN      |  MCU   | Direction |
|----------|--------|-----------|
|  TX      |  P0.06 |    out    |
|  RX      |  P0.08 |    in     |
| RTS      |  P0.05 |    out    |
| CTS      |  P0.07 |    in     |
| Time Sync|  P1.01 |    out    |

### HCI over UART 0 connected to first Virtual UART in J-Link Probe

Release build:
```sh
west build --pristine -b nrf52833dk/nrf52833
```
Debug build:
```sh
west build --pristine -b nrf52833dk/nrf52833 -- -DEXTRA_CONF_FILE=debug.conf
```

To use UART 0 via Arduino headers, the virtual UART of the J-Link probe needs to be disabled, e.g. with the JLink Configuration Tool.



## nRF5340 Development Kit

The first  Virtual UART (UART1, ...) is Zephyr UART 1
The second Virtual UART (UART2, ...) is Zephyr UART 0

### Pinout

Signal direction as seen from the nRF5340.

| PIN      | Arduino | MCU   | Direction |
|----------|---------|-------|-----------|
|  TX      |    D0   | P1.00 |    out    |
|  RX      |    D1   | P1.01 |    in     |
| RTS      |    D7   | P1.11 |    out    |
| CTS      |    D8   | P1.10 |    in     |
| Time Sync|    D10  | P1.06 |    out    |

### HCI over USB CDC

```sh
west build --pristine -b nrf5340dk/nrf5340/cpuapp -- -DEXTRA_DTC_OVERLAY_FILE=usb.overlay -DEXTRA_CONF_FILE=overlay-usb.conf
```


### HCI over UART 0 connected to second Virtual UART in J-Link Probe 
```sh
west build --pristine -b nrf5340dk/nrf5340/cpuapp
```

- HCI over second Virtual port / UART 0
- Boot banner on Arduino Header UART (P1.01) / UART 1 


### HCI over UART 1 connected to first Virtual UART in J-Link Probe as well as Arduino Headers
```sh
west build --pristine -b nrf5340dk/nrf5340/cpuapp -- -DEXTRA_DTC_OVERLAY_FILE=uart1.overlay
```

- No Boot Banner on Arduino Header UART



## nRF5340 Audio DK

The first UART (UART1, P1.04, P1.05) is only available via test points (TP60, TP61, ...)
The second UART (UART2, P1.08, P1.09) is connected to the Arduino headers.

### Pinout

Signal direction as seen from the nRF5340.

| PIN      | Arduino | MCU   | Direction |
|----------|---------|-------|-----------|
|  TX      |    D0   | P1.09 |    out    |
|  RX      |    D1   | P1.08 |    in     |
| RTS      |    D7   | P1.11 |    out    |
| CTS      |    D8   | P1.10 |    in     |
| Time Sync|    D10  | P1.01 |    out    |


### HCI over USB CDC

```sh
west build --pristine -b nrf5340_audio_dk/nrf5340/cpuapp -- -DEXTRA_DTC_OVERLAY_FILE=usb.overlay -DEXTRA_CONF_FILE=overlay-usb.conf
```

### HCI over UART 0 connected to Virtual UART in J-Link Probe

```sh
west build --pristine -b nrf5340_audio_dk/nrf5340/cpuapp
```

### HCI over UART 1 connected to Virtual UART in J-Link Probe as well as Arduino Headers

Release build:
```sh
west build --pristine -b nrf5340_audio_dk/nrf5340/cpuapp -- -DEXTRA_DTC_OVERLAY_FILE=uart1.overlay
```

Debug build:
```sh
west build --pristine -b nrf5340_audio_dk/nrf5340/cpuapp -- -DEXTRA_DTC_OVERLAY_FILE=uart1.overlay -DEXTRA_CONF_FILE=debug.conf
```

To use UART 1 via Arduino headers, the virtual UART of the J-Link probe needs to be disabled, e.g. with the JLink Configuration Tool.



## nRF54L15

### HCI over UART 1 connected to second Virtual UART in J-Link Probe

In Zephyr, the following DTS entry select the UART: `zephyr,bt-c2h-uart = &uart20`.
(It's not clear, why it's called uart20, but data is routed over UART 1 w.r.t to nRF4L15 Hardware User Guide)

Release build:
```sh
west build --pristine -b nrf54l15dk/nrf54l15/cpuapp
```
Debug build:
```sh
west build --pristine -b nrf54l15dk/nrf54l15/cpuapp -- -DEXTRA_CONF_FILE=debug.conf
```

To use UART 1 via pin headers, the virtual UARTs of the J-Link probe need to be disabled, e.g. with the JLink Configuration Tool.
As you might want to change the VDDM voltage, see next section, you can also use the Board Configurator GUI to disable VCOM1 and set VDDM.

### nRF54L15 Power Supply
Warning: On the nRF45L15 DK, the VDDM for the nRF54L15 is set to 1.8V by default!

Options to use it with 3.3V Bluetooth Host:
- Use the Board Configurator GUI, which is part of nRF Connect for Desktop, to change the VDDM to 3.3V
- remove the jumper from VDDM Current Measurement connector (P6) and connect 3.3V to VDDM (middle pin).

### Pinout

Signal direction as seen from the nRF54L15.

| PIN       | MCU   | Direction |
|-----------|-------|-----------|
| TX        | P1.04 |    out    |
| RX        | P1.05 |    in     |
| RTS       | P1.06 |    out    |
| CTS       | P1.07 |    in     |
| Time Sync | P1.11 |    out    |


## Maintainer Notes
- SoftDevice / Controller configuration for all targets are in `sybuild/ipc_radio.conf`.
- We can detect nRF5340 SoC in CMake with `if(CONFIG_SOC STREQUAL "nrf5340")` after find_package zephyr.


