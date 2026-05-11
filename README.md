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

ORCS is intended to be an open and commercially usable standard.

The recommended licensing approach for this project is a custom attribution-based specification license, inspired by permissive open standards models, but with a specific attribution requirement for the ORCS USB configuration interface.

This license is designed to allow broad adoption while preserving Neubras as the original creator and maintainer of the ORCS standard.

### Proposed license name

```text
ORCS Attribution License 1.0
```

### Proposed license text

```text
ORCS Attribution License 1.0

Copyright (c) Neubras.

Permission is hereby granted, free of charge, to any person or organization obtaining a copy of the OpenReader Config Standard (ORCS) specification to use, implement, reproduce, distribute, publish, reference, and create compatible implementations of the specification for commercial and non-commercial purposes, subject to the conditions below.

1. Attribution

Any implementation that claims compatibility with ORCS must preserve attribution to Neubras as the original creator and maintainer of the ORCS standard.

2. USB configuration descriptor attribution

Devices that implement the ORCS USB configuration mode must include a clear reference to Neubras in the USB descriptor associated with the ORCS configuration interface.

Recommended descriptor strings include one of the following:

"ORCS by Neubras"
"OpenReader Config Standard - Neubras"
"ORCS Configuration Interface - Neubras"

This attribution requirement applies only to the ORCS configuration interface.

3. Manufacturer independence

This license does not require manufacturers to change their own company name, product name, brand, enclosure, hardware design, firmware ownership, commercial identity, normal operation USB descriptors, or normal operation VID/PID.

Manufacturers may continue to use their own branding and device identification in normal operation mode.

4. No hardware restriction

ORCS does not define or restrict hardware design.

Any manufacturer may implement ORCS using its own electronics, firmware, RFID/NFC chipset, enclosure, communication interface, and product architecture.

5. Commercial use

Commercial use is explicitly allowed.

Manufacturers, integrators, software vendors, and service providers may use ORCS in commercial products, commercial software, embedded devices, configuration tools, access control systems, industrial systems, and related solutions.

6. Modifications and extensions

Implementations may add vendor-specific parameters, extensions, or private features, provided that they do not falsely represent those extensions as mandatory parts of the official ORCS specification.

If a modified version of the ORCS specification is published, it must clearly state that it is modified and must not be presented as the official ORCS specification unless approved by Neubras.

7. No warranty

The ORCS specification is provided "as is", without warranty of any kind, express or implied, including but not limited to warranties of merchantability, fitness for a particular purpose, and non-infringement.

In no event shall Neubras or contributors be liable for any claim, damages, or other liability arising from the use or implementation of this specification.
```

### License rationale

This license model was chosen because ORCS is not only software. It is mainly a technical specification for device configuration interoperability.

Traditional software licenses such as MIT, BSD, or Apache 2.0 are excellent for source code, libraries, SDKs, and tools, but they do not directly address the need for device-level attribution inside a USB configuration descriptor.

Creative Commons licenses are useful for documentation, but some variants, such as CC BY-NC-ND 4.0, are not suitable for this project because they restrict commercial use and modifications.

The ORCS Attribution License 1.0 is better aligned with the project goals because it:

* Allows commercial use.
* Allows implementation by different manufacturers.
* Does not restrict hardware design.
* Does not require manufacturers to rebrand their products.
* Requires attribution to Neubras only in the ORCS configuration interface.
* Protects the identity of the official ORCS standard.
* Allows vendor-specific extensions without breaking compatibility.

### Recommended licensing split

Because the project may include specifications, examples, schemas, tools, and reference software, the repository may use more than one license:

| Project part                          | Recommended license                       |
| ------------------------------------- | ----------------------------------------- |
| ORCS specification documents          | ORCS Attribution License 1.0              |
| JSON schemas and protocol definitions | ORCS Attribution License 1.0              |
| Example configuration files           | ORCS Attribution License 1.0 or CC BY 4.0 |
| Reference software tools              | Apache License 2.0 or MIT License         |
| SDKs and libraries                    | Apache License 2.0 or MIT License         |
| Documentation website                 | CC BY 4.0 or ORCS Attribution License 1.0 |

This separation allows the standard itself to preserve the required Neubras attribution while keeping software tools easy to adopt in commercial and open source ecosystems.

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
