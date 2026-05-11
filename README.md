# OpenReader Config Standard (ORCS)

**OpenReader Config Standard (ORCS)** is an open source specification for standardizing the configuration mode of RFID/NFC proximity card readers.

The goal of this project is to make reader configuration interoperable across different software, systems, manufacturers, and devices.

ORCS does **not** define a specific hardware design, electronic circuit, firmware architecture, or physical reader model. Instead, it defines a common configuration mode that compatible readers can implement so that any software can detect, configure, and manage them in a predictable way.

---

## Why ORCS exists

RFID/NFC proximity card readers are widely used in access control, automation, identification, industrial systems, commercial applications, and IoT solutions. However, each manufacturer usually provides its own configuration tool, proprietary protocol, or undocumented parameter format.

This creates several problems:

* Software integrations become dependent on specific reader brands or models.
* Developers need to implement different configuration methods for each device.
* Replacing a reader may require changes in the software.
* Installers and integrators often depend on proprietary tools.
* Configuration errors are common when output formats are not standardized.

ORCS aims to solve this by defining a simple, open, and extensible configuration standard for proximity card readers.

---

## Project objective

The objective of ORCS is to define a standard way for software applications to configure compatible RFID/NFC readers.

A compatible reader should expose a known configuration mode that allows software to read and write parameters such as:

* Output format
* Decimal or hexadecimal output
* Wiegand output configuration
* Number of bytes
* Byte order
* Bit order
* Prefix and suffix
* USB VID/PID
* HID keyboard mode
* Serial mode
* USB HID mode
* Card data formatting rules
* Device identification
* Firmware information
* Configuration version

This makes it possible for any compatible software to configure any compatible reader without requiring a proprietary configuration utility.

---

## What ORCS is not

ORCS is **not** a hardware standard.

This project does not define:

* The electronic design of the reader
* The antenna design
* The microcontroller or processor to be used
* The RFID/NFC chip to be used
* The enclosure or mechanical design
* The communication technology used to read the card
* The internal firmware architecture
* The card reading algorithm

Manufacturers are free to design their hardware and firmware as they prefer.

ORCS only defines the **configuration mode** and the expected behavior for reading and writing configuration parameters.

---

## Configuration mode

To support ORCS, a reader should provide a dedicated configuration mode.

When the device enters configuration mode, it should identify itself using a specific USB Vendor ID and Product ID reserved for ORCS-compatible configuration.

Recommended configuration mode identification:

```text
VID: XXXX
PID: YYYY
```

When operating normally, the reader may use its own manufacturer VID/PID or any device-specific identification. However, when it enters ORCS configuration mode, it should expose the standardized VID/PID so that configuration software can detect it automatically.

This allows software to distinguish between:

1. **Normal operation mode**
   The reader behaves as a keyboard, serial device, HID device, Wiegand interface, or another supported output mode.

2. **Configuration mode**
   The reader exposes the ORCS configuration interface using the standardized VID/PID.

---

## Why use a dedicated VID/PID for configuration

Using a dedicated VID/PID in configuration mode makes device discovery easier and more reliable.

A configuration tool can scan connected USB devices and identify ORCS-compatible readers by checking for:

```text
VID: XXXX
PID: YYYY
```

Once detected, the software can open the configuration interface and read or write supported parameters.

This avoids the need for:

* Manual device selection
* Brand-specific drivers
* Model-specific detection logic
* Proprietary discovery protocols
* Hardcoded device lists

The same software can configure readers from different manufacturers as long as they implement the ORCS configuration mode.

---

## Suggested operation flow

A typical ORCS-compatible configuration flow may work as follows:

1. The reader operates normally using its configured output mode.
2. The user or software requests the device to enter configuration mode.
3. The device reconnects or exposes a configuration interface using:

```text
VID: XXXX
PID: YYYY
```

4. The configuration software detects the ORCS-compatible reader.
5. The software reads the current configuration.
6. The user or system changes the desired parameters.
7. The software writes the new configuration to the reader.
8. The reader validates and stores the configuration.
9. The device exits configuration mode and returns to normal operation.

---

## Example configuration parameters

An ORCS-compatible reader may support parameters such as the following:

| Parameter            | Description                                                                                      |
| -------------------- | ------------------------------------------------------------------------------------------------ |
| `output_format`      | Defines the card output format, such as HEX, decimal, Wiegand, raw bytes, or custom format.      |
| `byte_length`        | Defines how many bytes of card data should be used.                                              |
| `byte_order`         | Defines whether the output should use big-endian or little-endian order.                         |
| `bit_order`          | Defines whether bits should be read in normal or reversed order.                                 |
| `prefix`             | Optional characters sent before the card number.                                                 |
| `suffix`             | Optional characters sent after the card number, such as Enter, Tab, CR, or LF.                   |
| `interface_mode`     | Defines whether the reader works as HID keyboard, USB serial, USB HID, Wiegand, or another mode. |
| `vid`                | USB Vendor ID used in normal operation mode, if configurable.                                    |
| `pid`                | USB Product ID used in normal operation mode, if configurable.                                   |
| `wiegand_bits`       | Defines the number of bits used in Wiegand output, such as 26, 34, 37, or custom.                |
| `facility_code`      | Optional facility code handling rule.                                                            |
| `card_number_format` | Defines how card data should be converted into the final output.                                 |
| `read_timeout`       | Defines timing behavior when reading card data.                                                  |
| `firmware_version`   | Returns the firmware version of the reader.                                                      |
| `config_version`     | Returns the supported ORCS configuration specification version.                                  |

The specification may evolve to include required, optional, and manufacturer-specific parameters.

---

## Example JSON configuration

The following example shows how a reader configuration could be represented:

```json
{
  "orcs_version": "1.0",
  "device": {
    "manufacturer": "Example Manufacturer",
    "model": "Example Reader",
    "firmware_version": "1.0.0"
  },
  "configuration": {
    "interface_mode": "hid_keyboard",
    "output_format": "decimal",
    "byte_length": 4,
    "byte_order": "big_endian",
    "bit_order": "normal",
    "prefix": "",
    "suffix": "enter",
    "wiegand_bits": 26
  }
}
```

This JSON is only an example. The final specification may define a binary protocol, JSON protocol, HID reports, serial commands, or multiple transport options.

---

## Transport layer

ORCS may support one or more transport layers for configuration, such as:

* USB HID reports
* USB CDC serial
* Vendor-defined USB interface
* BLE GATT service
* TCP/IP local network protocol
* NFC-based configuration

The first goal of the project is to define the configuration model and expected behavior. Transport-specific implementations may be defined as separate profiles.

For example:

* ORCS over USB HID
* ORCS over USB Serial
* ORCS over BLE
* ORCS over TCP/IP

---

## Compatibility concept

A device may be considered ORCS-compatible if it:

1. Provides a documented configuration mode.
2. Exposes the standard configuration VID/PID when using USB configuration mode.
3. Allows software to read supported configuration parameters.
4. Allows software to write supported configuration parameters.
5. Reports unsupported parameters clearly.
6. Preserves configuration after restart, when applicable.
7. Identifies the ORCS specification version it supports.

---

## Extensibility

ORCS should be extensible so that manufacturers can add custom parameters without breaking compatibility.

A suggested approach is to separate parameters into categories:

* **Core parameters**: common parameters expected by most readers.
* **Optional parameters**: supported by some readers depending on hardware capabilities.
* **Vendor-specific parameters**: custom features implemented by specific manufacturers.

Software should be able to query which parameters are supported before attempting to configure them.

---

## Example use cases

ORCS can be used in several scenarios:

* Access control systems
* Time attendance systems
* Industrial machine access control
* Commercial automation
* POS identification
* IoT devices
* Smart lockers
* Membership systems
* Visitor management systems
* Embedded Linux gateways
* Windows configuration tools
* Mobile configuration applications

---

## Benefits

### For software developers

* One configuration interface for multiple reader brands
* Less device-specific code
* Easier integration and testing
* Better compatibility across projects

### For manufacturers

* Easier adoption by software vendors
* Reduced need to maintain proprietary tools
* Clear compatibility with open ecosystems
* Better documentation and support experience

### For integrators and installers

* Faster setup
* Fewer configuration mistakes
* Easier device replacement
* Standard behavior across different readers

### For end users

* More freedom to choose hardware
* Less vendor lock-in
* More reliable system integration
* Longer device lifecycle

---

## Proposed repository structure

```text
orcs/
├── README.md
├── SPECIFICATION.md
├── examples/
│   ├── config-example.json
│   └── hid-report-example.md
├── profiles/
│   ├── usb-hid.md
│   ├── usb-serial.md
│   └── ble.md
├── schemas/
│   └── orcs-config.schema.json
└── tools/
    └── README.md
```

---

## Future goals

Possible future goals for ORCS include:

* Define the first stable configuration specification.
* Create JSON schemas for configuration files.
* Define USB HID report structures.
* Define serial command examples.
* Create reference firmware examples.
* Create a desktop configuration tool.
* Create a command-line configuration tool.
* Create test tools for compatibility validation.
* Publish certification guidelines for ORCS-compatible readers.

---

## Contributing

Contributions are welcome.

This project is intended to be developed collaboratively with software developers, hardware manufacturers, firmware engineers, system integrators, and access control professionals.

You can contribute by:

* Suggesting configuration parameters
* Proposing protocol formats
* Reviewing the specification
* Adding examples
* Implementing reference tools
* Testing with real readers
* Improving documentation

---

## License

This project should use an open source license that allows broad adoption by manufacturers and software developers.

Suggested licenses:

* MIT License
* Apache License 2.0
* BSD 3-Clause License

The final license should be chosen according to the project goals and contribution model.

---

## Summary

ORCS is an open standard for configuring RFID/NFC proximity card readers.

It does not define the hardware. It defines a common configuration mode.

By using a standardized configuration interface and a dedicated configuration VID/PID:

```text
VID: XXXX
PID: YYYY
```

any compatible software can detect and configure any compatible reader, regardless of manufacturer or hardware design.

The main goal is interoperability: one standard configuration method for many readers, systems, and use cases.
