# Velbus packet protocol
A guide to explain how the packet protocol works and how it's structured.


## Packet construction
---
Each packet is split up in three sections.

![Packet construction](https://github.com/velbus/packetprotocol/raw/master/img/packet.jpg "Packet construction")

The smallest possible packet has 6 bytes; 4 header bytes, 0 body bytes and 2 tail bytes.

The largest possible packet has 14 bytes; 4 header bytes, 8 body bytes and 2 tail bytes.

### Header section
---
The header is the preamble of the packet and consists of 4 bytes. All of the bytes in the header are mandatory.

![Header section](https://github.com/velbus/packetprotocol/raw/master/img/header.jpg "Header section")

#### Start flag
This byte marks the start of the packet and should **always** be 0x0F.

#### Priority flag
There are four different bytes assigned for priority, the table below lists them according to importance.

| Importance | Type  | Byte  |
| :-: |---|:-:|
| 1 | High  | 0xF8  |
| 2 | Firmware  | 0xF9  |
| 3 | Third party  | 0xFA  |
| 4 | Low  | 0xFB  |

If there are colliding packets on the bus, the one with the higher importance will get precedence.

#### Address
A packet always contains an address and is used to address a specific module.

If the packet is meant to be broadcast across every module, set the address to 0x00. This is why using the address 0x00 as a module address is not recommended.

#### RTR and body length
The RTR and body length byte is contained in one byte, and is split up in two parts. 

![RTR and body length](https://github.com/velbus/packetprotocol/raw/master/img/rtr-bodylength.jpg "RTR and body length")

#### RTR (Remote Transmit Request)
The high nibble is used in a packet in which it requests information without supplying any data (e.g. scanning the bus).

Should you wish to use RTR in the packet, mask the byte with 0x40.

#### Body length
The low nibble consists of the body length.

The maximum length of a Velbus packet is limited to 8 bytes, the document will go over more about this in the [Body section](#Body section).

### Body section
---
The body marks the payload of the packet and can consist of up to 8 bytes, but isn't required to have any data bytes at all.

![Body section](https://github.com/velbus/packetprotocol/raw/master/img/body.jpg "Body section")

Usually in a packet, the first byte is used to indicate the command of the packet, check the [Examples](#Examples) to get some concrete examples.

### Tail section
---
The tail (or end of the packet) marks the end of the packet and consists of 2 bytes. All of the bytes in the tail are mandatory.

![Tail section](https://github.com/velbus/packetprotocol/raw/master/img/tail.jpg "Tail section")

#### Checksum
This is the CRC8 checksum of all of the previous bytes.

To calculate the checksum, take the sum all of the previous bytes as an unsigned byte, then perform the two's compliment on it.

#### End flag
This byte marks the end of the packet and should **always** be 0x04.

## Examples
---

### Scan
> **Important** 
>
> The address 0x00 in this case is not meant as a broadcast!

Construct a packet with the following properties
* [Low priority](#Priorityflag)
* Set the requested address
* Set the RTR mask (no data length)

Example for address 0x06

![Scan packet](https://github.com/velbus/packetprotocol/raw/master/img/scan-packet.jpg "Scan packet")

### Swich relay on

Construct a packet with the following properties
* [High priority](#Priorityflag) 
* Set the requested address
* No RTR, two bytes data length
* Set the first data byte (command) to 0x02 "Switch relay on"
* Set the second data byte to the channel number

Example for 4RYLD with address 0x0B, channels number 2&3

![Switch relay on](https://github.com/velbus/packetprotocol/raw/master/img/relay-switch-on-packet.jpg "Switch relay on")

### Write data to the memory map

Using the command 0xCA, it is possible to write a sequence (block) of 4 bytes in one command.

> **Important**
>
> Not every module supports the 0xCA command and has the same layout and size of the memory map, make sure you consult each individual protocol manual!

Construct a packet with the following properties
* [Low priority](#Priorityflag)
* Set the requested address
* No RTR, 7 bytes data length
* Set the first data byte (command) to 0xCA "Write memory block"
* Set the second data byte to the high memory address
* Set the third data byte to the low memory address
* Set the fourth data byte to the first databyte to write
* Set the fifth data byte to the second databyte to write
* Set the sixth data byte to the third databyte to write
* Set the seventh data byte to the fourth databyte to write


Example for 4RYLD
* Module address 0x4D
* Memory address 0x00E4 (4 bytes so it extends to 0x00E7)
* First four letters of the module name (skipping one since the first letter falls under the previous memory address)

![Write data block](https://github.com/velbus/packetprotocol/raw/master/img/write-data-block-packet.jpg "Write data block")