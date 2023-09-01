# Ricoh Type 145 Toner Chip Investigations
I currently own a Ricoh C420DN Color Laser printer which uses Type 145 toner.  As new toner can be quite expensive, I've been turing to the used overstock markets to buy toner.  Along the way I've been able to purchase new old stock toner, as well as used toner.

I've noticed that some of the previous used toner, when placed into the printer, doesn't reset the supply level.  Searching the internet for information regarding how the supply level works led me to the chip embedded into the toner cartridge itself.  

Since I had an empty cartridge lying around, I pulled the chip off to examine it:


The interface:
![Image of board interface](images/ricohtype145boardinterface.jpg)
The face:
![Image of board face](images/ricohtype145boardface.jpg)

# Pin Identification
![Image of Pin Identification](images/ricohtype145boardident.jpg)
TSSOP8 looks like an EEPROM with I2C interface.  Signal identification was fairly easy - as two pads on the interface side go directly to IC through resistors.  The other two pads must be Vcc/Ground.  Ground was identified by the larger copper traces, along with direct connection to the IC.  Which leaves Vcc, which is connected to Vcc with a capacitor, and a couple of address lines via 10K resistors.  

The address line settings aligns with black toner cartridge I2C address space. SCL/SDA are passed with vias to the connector side of the board.  
![Image of Pin Interface](images/ricohtype145boardinterfaceident.jpg)

# Interfacing 
A helpful tool for interfacing with unknown busses is the [Bus Pirate](http://dangerousprototypes.com/docs/Bus_Pirate) 

I used it for this investigation.  Using a v3b version. 

# Bus Pirate Interface
![Image of Bus Pirate Pins](images/Bp-cable-color.png)
Using the above map
| Bus Pirate Pin | Destination |
| :--------------: | ----------- |
| GND            | GND on Toner board |
| 3V3            | Vcc on Toner board |
| VPU            | 3V3 on Bus Pirate |
| MOSI           | SDA on Toner board |
| CLK            | SCL on Toner board |

# Bus Pirate Configuration
<ol>
<li>Connect to toner board - jumpers and tape work well here</li>
<li>Switch to I2C mode

```
HiZ>m
1. HiZ
2. 1-WIRE
3. UART
4. I2C
5. SPI
6. 2WIRE
7. 3WIRE
8. LCD
9. DIO
x. exit(without change)

(1)>4
```
</li>
<li>400KHz speed

```
Set speed:
 1. ~5KHz
 2. ~50KHz
 3. ~100KHz
 4. ~400KHz

(1)>4
Ready
```
</li>
<li>Switch power supply on

```
I2C>W
Power supplies ON
```
</li>
<li>Validate VPU@3.3V

```
I2C>v
Pinstates:
1.(BR)  2.(RD)  3.(OR)  4.(YW)  5.(GN)  6.(BL)  7.(PU)  8.(GR)  9.(WT)  0.(Blk)
GND     3.3V    5.0V    ADC     VPU     AUX     SCL     SDA     -       -
P       P       P       I       I       I       I       I       I       I
GND     3.28V   4.97V   0.00V   3.28V   L       L       L       L       L
```
</li>
<li>Set pullup resistors to on

```
I2C>P
Pull-up resistors ON
```
</li>
<li>Search for I2C devices

```
I2C>(1)
Searching I2C address space. Found devices at:
0xA6(0x53 W) 0xA7(0x53 R)
```
</li>
</ol>

# Toner EEPROM Address list

| Address | Color |
| ------- | ----- |
| 0xA0(0x50 W) 0xA1(0x50 R) | Yellow |
| 0xA2(0x51 W) 0xA3(0x51 R) | Magenta |
| 0xA4(0x52 W) 0xA5(0x52 R) | Cyan |
| 0xA6(0x53 W) 0xA7(0x53 R) | Black |

# Reading Toner EEPROM
Once the toner board is connected to the Bus Pirate, use a read command to read the EEPROM.  I found that for the Type 145 boards that I have, reading past 128 bytes repeats data.
```
I2C>[0xa6 0x00 [0xa7 r:128]
I2C START BIT
WRITE: 0xA6 ACK
WRITE: 0x00 ACK
I2C START BIT
WRITE: 0xA7 ACK
READ: 0xA2  ACK 0x00  ACK 0x01  ACK 0x02  ACK 0x0F  ACK 0x01  ACK 0x01  ACK 0xFF  ACK 0x00  ACK 0x00  ACK 0xFF  ACK 0xFF  ACK 0xFF  ACK 0xFF  ACK 0xFF  ACK 0xFF  ACK 0x32  ACK 0x30  ACK 0x31  ACK 0x33  ACK 0x30  ACK 0x34  ACK 0x4B  ACK 0x4A  ACK 0x43  ACK 0x37  ACK 0x30  ACK 0x30  ACK 0x33  ACK 0x36  ACK 0x38  ACK 0x32  ACK 0x38  ACK 0x38  ACK 0x38  ACK 0x33  ACK 0x30  ACK 0x38  ACK 0x00  ACK 0x00  ACK 0x0E  ACK 0x00  ACK 0x00  ACK 0x00  ACK 0xFF  ACK 0xFF  ACK 0xFF  ACK 0xFF  ACK 0x00  ACK 0x00  ACK 0x87  ACK 0xA5  ACK 0x00  ACK 0x02  ACK 0xE3  ACK 0x4A  ACK 0xFF  ACK 0xFF  ACK 0xFF  ACK 0xFF  ACK 0xFF  ACK 0xFF  ACK 0xFF  ACK 0xFF  ACK 0x00  ACK 0xFF  ACK 0xFF  ACK 0xFF  ACK 0xFF  ACK 0xFF  ACK 0xFF  ACK 0xFF  ACK 0xFF  ACK 0xFF  ACK 0xFF  ACK 0xFF  ACK 0xFF  ACK 0xFF  ACK 0xFF  ACK 0xFF  ACK 0xFF  ACK 0xFF  ACK 0xFF  ACK 0xFF  ACK 0xFF  ACK 0xFF  ACK 0xFF  ACK 0xFF  ACK 0x20  ACK 0x06  ACK 0x01  ACK 0x22  ACK 0xFF  ACK 0xFF  ACK 0xFF  ACK 0xFF  ACK 0xFF  ACK 0xFF  ACK 0xFF  ACK 0xFF  ACK 0xFF  ACK 0xFF  ACK 0xFF  ACK 0xFF  ACK 0xFF  ACK 0xFF  ACK 0xFF  ACK 0xFF  ACK 0xFF  ACK 0xFF  ACK 0xFF  ACK 0xFF  ACK 0x43  ACK 0x48  ACK 0x49  ACK 0x50  ACK 0xFF  ACK 0xFF  ACK 0xFF  ACK 0xFF  ACK 0xFF  ACK 0xFF  ACK 0xFF  ACK 0xFF  ACK 0xFF  ACK 0xFF  ACK 0xFF  ACK 0xFF
NACK
I2C STOP BIT
I2C>
```

You can then cut/paste the READ output into a text file, and then convert to binary using:
```
sed -i -e "s/  ACK//g" output.txt ; cat blah.txt | xargs printf "%02x\n" | xxd -c 1 -p -r > output.bin
```
Then display the binary in Hex/ASCII:
```
xxd blah.bin
00000000: a200 0102 0f01 01ff 0000 ffff ffff ffff  ................
00000010: 3230 3133 3034 4b4a 4337 3030 3336 3832  201304KJC7003682
00000020: 3838 3833 3038 0000 0e00 0000 ffff ffff  888308..........
00000030: 0000 87a5 0002 e34a ffff ffff ffff ffff  .......J........
00000040: 00ff ffff ffff ffff ffff ffff ffff ffff  ................
00000050: ffff ffff ffff ffff 2006 0122 ffff ffff  ........ .."....
00000060: ffff ffff ffff ffff ffff ffff ffff ffff  ................
00000070: 4348 4950 ffff ffff ffff ffff ffff ffff  CHIP............
```

Or you can use [i2c-dump](https://github.com/AdamLaurie/i2c-dump/tree/master) to dump the EEPROM to a binary file. 
# EEPROM Analysis
0x00 - 0x07 : header values
```
00000000: a200 0102 0301 03ff  | Black 145
00000000: a200 0102 0f01 01ff  | Black 145 HY
00000000: a200 0102 0302 03ff  | Cyan 145 
00000000: a200 0102 0f02 01ff  | Cyan 145 HY
00000000: a200 0102 0303 03ff  | Magenta 145 
00000000: a200 0102 0f03 01ff  | Magenta 145 HY
00000000: a200 0102 0304 03ff  | Yellow 145 
00000000: a200 0102 0f04 01ff  | Yellow 145 HY
```
* Note: The values listed as 145 non-HY variants I can't verify as I don't have the origional packaging for them

0x08 - 0x09 : New/Installed Cartridge?
```
6400 | New Cartridge
0000 | Installed Cartridge
```
0x10 - 0x15 : YYYYMM - Cartridge Manufacture Year/Month

0x16 - 0x19 : Static 
```
00000016: 4b4a 4337 
```
0x1a - 0x1f : Batch Production number
```
0000001a: 3030 3336 3832  | 003682
```

0x20 - 0x27 - EDP Code 
```
00000020: 3539 3333 3331 0000 | 593331 | Black ?
00000020: 3838 3833 3038 0000 | 888308 | Black 145 HY
00000020: 3539 3333 3332 0000 | 593332 | Yellow ?
00000020: 3838 3833 3039 0000 | 888309 | Yellow 145 HY
00000020: 3539 3333 3333 0000 | 593333 | Magenta ?
00000020: 3838 3833 3130 0000 | 888310 | Magenta 145 HY
00000020: 3539 3333 3334 0000 | 593334 | Cyan ?
00000020: 3838 3833 3131 0000 | 888311 | Cyan 145 HY
```
* Note: I can't validate the non-HY EDP codes.  I assume they are non-HY (high yield), but there are 888 based EDP codes for those - 888276:Black, 888277:Yellow, 888278:Magenta, 888279:Cyan

0x28 - 0x29 - New/Used Cartridge ?
```
6400 | New Cartridge
004e | Installed Cartridge
0a4e | Installed Cartridge
```

0x2a - 0x2f -
```
0000002a: 0000 ffff ffff
```

0x30 - 0x40 - Usage data?

0x41 - 0x58 - 0xff

0x58 - 0x5b - YYYYMMDD - Install Date of Cartridge (GMT)
```
00000058: 2023 0825
```
0x5c - 0x5f - YYYYMMDD - 2nd Install Date (GMT)?

0x70 - 0x73 - When installed set to 'CHIP'
```
00000070: 4348 4950 | CHIP
```

0x74 - 0x7e - Printer Machine ID
```
00000074: 53XX XXXX XXXX XXXX XXXX XX00 | SXXXXXXXXXX
```
* Note: Printer serial number doesn't appear on every installed cartridge.  

* New cartridges: 0x30 - 0x7f: 0xff

# Summary
The above data was gathered from 29 cartridges. A large group of them were new old stock. I'd like to take a look closer at the cartridges that aren't Type 145 HY - and dump more of those.  They are the ones that I picked up from Ebay used.

One other note, the ID printed on the cartridge is the batch number - I was able to confirm that due to getting a couple of cartridges from the same batch.  They have different batch production numbers that differ by 1.  

# Links
* [i2c-dump](https://github.com/AdamLaurie/i2c-dump/tree/master)
* [Toner Chip Reset](https://github.com/lugu/toner_chip_reset)
