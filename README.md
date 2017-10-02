# Velbus packet protocol
Read this guide to understand how the packet protocol works and how it is structured.

## Communication
The communication between Velbus modules happens on byte level, these bytes make up a packet.

Every Velbus module knows how to send and respond to these packets.

## Packet construction
Each packet is split up into three sections.

<table class="table table-bordered">
	<thead>
		<tr>
			<th colspan="3" align="center" width=400px">Packet</th>
		</tr>
		<tr>
			<th align="center" width="125px">Header</th>
			<th align="center" width="175px">Body</th>
			<th align="center" width="100px">Tail</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td align="center" width="125px">4 bytes</td>
			<td align="center" width="175px">0-8 bytes</td>
			<td align="center" width="100px">2 bytes</td>
		</tr>
	</tbody>
</table>

The smallest possible packet has 6 bytes; 4 header bytes, 0 body bytes and 2 tail bytes.

The largest possible packet has 14 bytes; 4 header bytes, 8 body bytes and 2 tail bytes.

### Header section
The header is the preamble of the packet and consists out of 4 bytes. All bytes in the header are mandatory.

<table class="table table-striped table-bordered">
	<thead>
		<tr>
			<th colspan="4" align="center" width="550px">Header</th>
		</tr>
		<tr>
			<th align="center" width="100px">Start flag</th>
			<th align="center" width="150px">Priority flag</th>
			<th align="center" width="100px">Address</th>
			<th align="center" width="200px">RTR and body length</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td align="center" width=100px">1 byte</td>
			<td align="center" width=150px">1 byte</td>
			<td align="center" width=200px">1 byte</td>
			<td align="center" width=100px">1 byte</td>
		</tr>
	</tbody>
</table>

#### Start flag
This byte marks the start of the packet and should **always** be **0x0F**.

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

If the packet is meant to be broadcast across every module, set the address to **0x00**. This is why using the address 0x00 as a module address is not recommended.

#### RTR and body length
The RTR and body length byte is contained in one byte, and is split up in two parts. 

<table class="table table-striped table-bordered">
	<thead>
		<tr>
			<th colspan="2" align="center" width=300px">RTR and body length</th>
		</tr>
		<tr>
			<th align="center" width=150px">High nibble</th>
			<th align="center" width=150px">Low nibble</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td align="center" width=150px">RTR</td>
			<td align="center" width=150px">Body length</td>
		</tr>
	</tbody>
</table>

#### RTR (Remote Transmit Request)
The high nibble is used in a packet in which it requests information without supplying any data (e.g. scanning the bus).

Should you wish to use RTR in the packet, mask the byte with **0x40**.

#### Body length
The low nibble consists out of the body length.

The maximum body length of a Velbus packet is limited to 8 bytes, the document will go into detail on this subject in the [Body section](#body-section).

### Body section
The body marks the payload of the packet and can consist out of up to 8 bytes, but isn't required to have any data bytes at all.

<table class="table table-striped table-bordered">
	<thead>
		<tr>
			<th colspan="2" align="center" width=100px">Body</th>
		</tr>
		<tr>
			<th align="center" width=100px">Data</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td align="center" width=100px">0-8 bytes</td>	
		</tr>
	</tbody>
</table>

Usually in a packet, the first byte is used to indicate the command of the packet, check the [examples](#examples) to get some concrete usages.

### Tail section
The tail marks the end of the packet and consists out of 2 bytes. All of the bytes in the tail are mandatory.

<table class="table table-striped table-bordered">
	<thead>
		<tr>
			<th colspan="2" align="center" width=200px">Tail</th>
		</tr>
		<tr>
			<th align="center" width=100px">Checksum</th>
			<th align="center" width=100px">End flag</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td align="center" width=100px">1 byte</td>
			<td align="center" width=100px">1 byte</td>
		</tr>
	</tbody>
</table>

#### Checksum
> **Important**
>
> Make sure that you are working with unsigned bytes while calculating the checksum!

To calculate the checksum, take the sum of all the previous bytes and then perform the 'two's compliment' on it.

#### End flag
This byte marks the end of the packet and should **always** be **0x04**.

## Examples

These examples only list a couple of commands that are available on the modules, for more information please refer to the [module protocol manuals](https://github.com/velbus/moduleprotocol).

### Scan
> **Important** 
>
> The address 0x00 in this case can not be used for broadcasting!

Construct a packet with the following properties
* [low priority](#priority-flag)
* set the requested address
* set the RTR mask (no data length)

Example for address 0x06

<table class="table table-striped table-bordered">
	<thead>
		<tr>
			<th colspan="4" align="center" width=280px">Header</th>
			<th colspan="2" align="center" width=140px">Tail</th>
		</tr>
		<tr>
			<th align="center" width=70px">STX</th>
			<th align="center" width=70px">PRIO</th>
			<th align="center" width=70px">ADDR</th>
			<th align="center" width=70px">RTR+L</th>
			<th align="center" width=70px">CRC</th>
			<th align="center" width=70px">ETX</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td align="center" width=70px">0x0F</td>
			<td align="center" width=70px">0xFB</td>
			<td align="center" width=70px">0x06</td>
			<td align="center" width=70px">0x40</td>
			<td align="center" width=70px">0xB0</td>
			<td align="center" width=70px">0x04</td>
		</tr>
	</tbody>
</table>

### Switch relay on

Construct a packet with the following properties
* [high priority](#priority-flag) 
* set the requested address
* no RTR, two bytes data length
* set the first data byte (command) to 0x02 "Switch relay on"
* set the second data byte to the channel number

Example for 4RYLD with address 0x0B, channel numbers 2 & 3

<table class="table table-striped table-bordered">
	<thead>
		<tr>
			<th colspan="4" align="center" width=280px">Header</th>
			<th colspan="2" align="center" width=140px">Body</th>
			<th colspan="2" align="center" width=140px">Tail</th>
		</tr>
		<tr>
			<th align="center" width=70px">STX</th>
			<th align="center" width=70px">PRIO</th>
			<th align="center" width=70px">ADDR</th>
			<th align="center" width=70px">RTR+L</th>
			<th align="center" width=70px">D1</th>
			<th align="center" width=70px">D2</th>
			<th align="center" width=70px">CRC</th>
			<th align="center" width=70px">ETX</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td align="center" width=70px">0x0F</td>
			<td align="center" width=70px">0xF8</td>
			<td align="center" width=70px">0x0B</td>
			<td align="center" width=70px">0x02</td>
			<td align="center" width=70px">0x02</td>
			<td align="center" width=70px">0x04</td>
			<td align="center" width=70px">0xE4</td>
			<td align="center" width=70px">0x04</td>
		</tr>
	</tbody>
</table>

### Write data to the memory map

By using the 0xCA command, it is possible to write a sequence (block) of 4 bytes in one command.

> **Important**
>
> Not every module supports the 0xCA command *and* has the same layout and size of the memory map so make sure that you consult each individual protocol manual!

Construct a packet with the following properties
* [low priority](#priority-flag)
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
* module address 0x4D
* memory address 0x00E4 (4 bytes so it extends to 0x00E7)
* first four letters of the module name (skipping one since the first letter falls under the previous memory address)

<table class="table table-striped table-bordered">
	<thead>
		<tr>
			<th colspan="4" align="center" width=280px">Header</th>
			<th colspan="7" align="center" width=490px">Body</th>
			<th colspan="2" align="center" width=140px">Tail</th>
		</tr>
		<tr>
			<th align="center" width=70px">STX</th>
			<th align="center" width=70px">PRIO</th>
			<th align="center" width=70px">ADDR</th>
			<th align="center" width=70px">RTR+L</th>
			<th align="center" width=70px">D1</th>
			<th align="center" width=70px">D2</th>
			<th align="center" width=70px">D3</th>
			<th align="center" width=70px">D4</th>
			<th align="center" width=70px">D5</th>
			<th align="center" width=70px">D6</th>
			<th align="center" width=70px">D7</th>
			<th align="center" width=70px">CRC</th>
			<th align="center" width=70px">ETX</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td align="center" width=70px">0x0F</td>
			<td align="center" width=70px">0xFB</td>
			<td align="center" width=70px">0x4D</td>
			<td align="center" width=70px">0x07</td>
			<td align="center" width=70px">0xCA</td>
			<td align="center" width=70px">0x00</td>
			<td align="center" width=70px">0xE4</td>
			<td align="center" width=70px">0x4D</td>
			<td align="center" width=70px">0x42</td>
			<td align="center" width=70px">0x34</td>
			<td align="center" width=70px">0x52</td>
			<td align="center" width=70px">0xDF</td>
			<td align="center" width=70px">0x04</td>
		</tr>
	</tbody>
</table>