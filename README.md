# Velbus packet protocol
Read this guide to understand how the packet protocol works and how it is structured.

## Communication
The communication between Velbus modules happens on byte level, these bytes make up a packet.

Every Velbus module knows how to send and respond to these packets.

## Packet construction
Each packet is split up into three sections.

![Packet construction](https://github.com/velbus/packetprotocol/raw/master/img/packet.jpg "Packet construction")

The smallest possible packet has 6 bytes; 4 header bytes, 0 body bytes and 2 tail bytes.

The largest possible packet has 14 bytes; 4 header bytes, 8 body bytes and 2 tail bytes.

### Header section
The header is the preamble of the packet and consists out of 4 bytes. All bytes in the header are mandatory.

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
The low nibble consists out of the body length.

The maximum body length of a Velbus packet is limited to 8 bytes, the document will go into detail on this subject in the [Body section](#Body-section).

### Body section
The body marks the payload of the packet and can consist out of up to 8 bytes, but isn't required to have any data bytes at all.

![Body section](https://github.com/velbus/packetprotocol/raw/master/img/body.jpg "Body section")

Usually in a packet, the first byte is used to indicate the command of the packet, check the [Examples section](#Examples) to get some concrete examples.

### Tail section
The tail marks the end of the packet and consists out of 2 bytes. All of the bytes in the tail are mandatory.

![Tail section](https://github.com/velbus/packetprotocol/raw/master/img/tail.jpg "Tail section")

#### Checksum
> **Important**
>
> Make sure that you are working with unsigned bytes while calculating the checksum!

To calculate the checksum, take the sum of all the previous bytes and then perform the 'two's compliment' on it.

#### End flag
This byte marks the end of the packet and should **always** be 0x04.

## Examples

These examples only list a couple of commands that are available on the modules, for more information please refer to the module protocol manuals.

### Scan
> **Important** 
>
> The address 0x00 in this case can not be used as a broadcast!

Construct a packet with the following properties
* [low priority](#Priority-flag)
* set the requested address
* set the RTR mask (no data length)

Example for address 0x06

![Scan packet](https://github.com/velbus/packetprotocol/raw/master/img/scan-packet.jpg "Scan packet")

### Switch relay on

Construct a packet with the following properties
* [high priority](#Priority-flag) 
* set the requested address
* no RTR, two bytes data length
* set the first data byte (command) to 0x02 "Switch relay on"
* set the second data byte to the channel number

Example for 4RYLD with address 0x0B, channels number 2&3

![Switch relay on](https://github.com/velbus/packetprotocol/raw/master/img/relay-switch-on-packet.jpg "Switch relay on")

### Write data to the memory map

By using the 0xCA command, it is possible to write a sequence (block) of 4 bytes in one command.

> **Important**
>
> Not every module supports the 0xCA command and has the same layout and size of the memory map so make sure that you consult each individual protocol manual!

Construct a packet with the following properties
* [low priority](#Priorityflag)
* set the requested address
* no RTR, 7 bytes data length
* set the first data byte (command) to 0xCA "Write memory block"
* set the second data byte to the high memory address
* set the third data byte to the low memory address
* set the fourth data byte to the first databyte to write
* set the fifth data byte to the second databyte to write
* set the sixth data byte to the third databyte to write
* set the seventh data byte to the fourth databyte to write

Example for 4RYLD
* Module address 0x4D
* Memory address 0x00E4 (4 bytes so it extends to 0x00E7)
* First four letters of the module name (skipping one since the first letter falls under the previous memory address)

![Write data block](https://github.com/velbus/packetprotocol/raw/master/img/write-data-block-packet.jpg "Write data block")