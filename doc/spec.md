This will eventually become a spec for the keyboard driver program.

What do I think I know so far?

# How To Reverse Engineer

The approach I'm using is running Windows 7 in a Virtual Box VM on Ubuntu 18.04,
with Wireshark capturing USB packets between the keyboard and the host.

It took a bit of fighting to get a working setup where Windows could see the
keyboard well enough for the driver program to work.

Creating a VM and installing Windows on it was easy.  That step just worked.
However, getting Windows to see the nonstandard USB interfaces of the keyboard
was harder.

One thing I had to do was tell virtualbox to use
/usr/share/virtualbox/VBoxGuestAdditions.iso as a storage device in Windows so
that I could install the windows parts of the USB drivers.

I also had to expose /usr/share/virtualbox-ext-pack to make VirtualBox have
the all the USB bridge drivers necessary.

To get the keyboard exposed, first I had to detach the device from linux so
that virtualbox could expose it to windows:

``
 1144  echo -n '3-11.3:1.1' > /sys/bus/usb/drivers/usbhid/unbind
 1146  echo -n '3-11.3:1.0' > /sys/bus/usb/drivers/usbhid/unbind
 1147  echo -n '3-11.3:1.2' > /sys/bus/usb/drivers/usbhid/unbind
``

At this point, I could install the xbows driver and it could see the keyboard
properly.  Then I could use Wireshark to capture USB packets going back and
forth.



# Basic Device USB HID info

Here is the detailed device descriptor.

```
root@cerberus:/home/jlquinn# lsusb -vd 1EA7:0907

Bus 003 Device 108: ID 1ea7:0907  
Device Descriptor:
  bLength                18
  bDescriptorType         1
  bcdUSB               1.10
  bDeviceClass            0 (Defined at Interface level)
  bDeviceSubClass         0 
  bDeviceProtocol         0 
  bMaxPacketSize0        64
  idVendor           0x1ea7 
  idProduct          0x0907 
  bcdDevice            3.00
  iManufacturer           1 SEMITEK
  iProduct                2 USB-HID Gaming Keyboard
  iSerial                 3 SN0000000001
  bNumConfigurations      1
  Configuration Descriptor:
    bLength                 9
    bDescriptorType         2
    wTotalLength           91
    bNumInterfaces          3
    bConfigurationValue     1
    iConfiguration          0 
    bmAttributes         0xa0
      (Bus Powered)
      Remote Wakeup
    MaxPower              100mA
    Interface Descriptor:
      bLength                 9
      bDescriptorType         4
      bInterfaceNumber        0
      bAlternateSetting       0
      bNumEndpoints           1
      bInterfaceClass         3 Human Interface Device
      bInterfaceSubClass      1 Boot Interface Subclass
      bInterfaceProtocol      1 Keyboard
      iInterface              0 
        HID Device Descriptor:
          bLength                 9
          bDescriptorType        33
          bcdHID               1.11
          bCountryCode            0 Not supported
          bNumDescriptors         1
          bDescriptorType        34 Report
          wDescriptorLength      64
          Report Descriptor: (length is 64)
            Item(Global): Usage Page, data= [ 0x01 ] 1
                            Generic Desktop Controls
            Item(Local ): Usage, data= [ 0x06 ] 6
                            Keyboard
            Item(Main  ): Collection, data= [ 0x01 ] 1
                            Application
            Item(Global): Usage Page, data= [ 0x07 ] 7
                            Keyboard
            Item(Local ): Usage Minimum, data= [ 0xe0 ] 224
                            Control Left
            Item(Local ): Usage Maximum, data= [ 0xe7 ] 231
                            GUI Right
            Item(Global): Logical Minimum, data= [ 0x00 ] 0
            Item(Global): Logical Maximum, data= [ 0x01 ] 1
            Item(Global): Report Size, data= [ 0x01 ] 1
            Item(Global): Report Count, data= [ 0x08 ] 8
            Item(Main  ): Input, data= [ 0x02 ] 2
                            Data Variable Absolute No_Wrap Linear
                            Preferred_State No_Null_Position Non_Volatile Bitfield
            Item(Global): Report Count, data= [ 0x01 ] 1
            Item(Global): Report Size, data= [ 0x08 ] 8
            Item(Main  ): Input, data= [ 0x03 ] 3
                            Constant Variable Absolute No_Wrap Linear
                            Preferred_State No_Null_Position Non_Volatile Bitfield
            Item(Global): Report Count, data= [ 0x03 ] 3
            Item(Global): Report Size, data= [ 0x01 ] 1
            Item(Global): Usage Page, data= [ 0x08 ] 8
                            LEDs
            Item(Local ): Usage Minimum, data= [ 0x01 ] 1
                            NumLock
            Item(Local ): Usage Maximum, data= [ 0x03 ] 3
                            Scroll Lock
            Item(Main  ): Output, data= [ 0x02 ] 2
                            Data Variable Absolute No_Wrap Linear
                            Preferred_State No_Null_Position Non_Volatile Bitfield
            Item(Global): Report Count, data= [ 0x01 ] 1
            Item(Global): Report Size, data= [ 0x05 ] 5
            Item(Main  ): Output, data= [ 0x03 ] 3
                            Constant Variable Absolute No_Wrap Linear
                            Preferred_State No_Null_Position Non_Volatile Bitfield
            Item(Global): Report Count, data= [ 0x06 ] 6
            Item(Global): Report Size, data= [ 0x08 ] 8
            Item(Global): Logical Minimum, data= [ 0x00 ] 0
            Item(Global): Logical Maximum, data= [ 0xa4 0x00 ] 164
            Item(Global): Usage Page, data= [ 0x07 ] 7
                            Keyboard
            Item(Local ): Usage Minimum, data= [ 0x00 ] 0
                            No Event
            Item(Local ): Usage Maximum, data= [ 0xa4 ] 164
                            ExSel
            Item(Main  ): Input, data= [ 0x00 ] 0
                            Data Array Absolute No_Wrap Linear
                            Preferred_State No_Null_Position Non_Volatile Bitfield
            Item(Main  ): End Collection, data=none
      Endpoint Descriptor:
        bLength                 7
        bDescriptorType         5
        bEndpointAddress     0x81  EP 1 IN
        bmAttributes            3
          Transfer Type            Interrupt
          Synch Type               None
          Usage Type               Data
        wMaxPacketSize     0x0008  1x 8 bytes
        bInterval               1
    Interface Descriptor:
      bLength                 9
      bDescriptorType         4
      bInterfaceNumber        1
      bAlternateSetting       0
      bNumEndpoints           2
      bInterfaceClass         3 Human Interface Device
      bInterfaceSubClass      0 No Subclass
      bInterfaceProtocol      0 None
      iInterface              0 
        HID Device Descriptor:
          bLength                 9
          bDescriptorType        33
          bcdHID               1.11
          bCountryCode            0 Not supported
          bNumDescriptors         1
          bDescriptorType        34 Report
          wDescriptorLength      34
          Report Descriptor: (length is 34)
            Item(Global): Usage Page, data= [ 0x00 0xff ] 65280
                            (null)
            Item(Local ): Usage, data= [ 0x50 ] 80
                            (null)
            Item(Main  ): Collection, data= [ 0x01 ] 1
                            Application
            Item(Local ): Usage, data= [ 0x02 ] 2
                            (null)
            Item(Global): Logical Minimum, data= [ 0x00 ] 0
            Item(Global): Logical Maximum, data= [ 0xff 0x00 ] 255
            Item(Global): Report Size, data= [ 0x08 ] 8
            Item(Global): Report Count, data= [ 0x40 ] 64
            Item(Main  ): Input, data= [ 0x02 ] 2
                            Data Variable Absolute No_Wrap Linear
                            Preferred_State No_Null_Position Non_Volatile Bitfield
            Item(Local ): Usage, data= [ 0x03 ] 3
                            (null)
            Item(Global): Logical Minimum, data= [ 0x00 ] 0
            Item(Global): Logical Maximum, data= [ 0xff 0x00 ] 255
            Item(Global): Report Size, data= [ 0x08 ] 8
            Item(Global): Report Count, data= [ 0x40 ] 64
            Item(Main  ): Output, data= [ 0x02 ] 2
                            Data Variable Absolute No_Wrap Linear
                            Preferred_State No_Null_Position Non_Volatile Bitfield
            Item(Main  ): End Collection, data=none
      Endpoint Descriptor:
        bLength                 7
        bDescriptorType         5
        bEndpointAddress     0x83  EP 3 IN
        bmAttributes            3
          Transfer Type            Interrupt
          Synch Type               None
          Usage Type               Data
        wMaxPacketSize     0x0040  1x 64 bytes
        bInterval               1
      Endpoint Descriptor:
        bLength                 7
        bDescriptorType         5
        bEndpointAddress     0x04  EP 4 OUT
        bmAttributes            3
          Transfer Type            Interrupt
          Synch Type               None
          Usage Type               Data
        wMaxPacketSize     0x0040  1x 64 bytes
        bInterval               1
    Interface Descriptor:
      bLength                 9
      bDescriptorType         4
      bInterfaceNumber        2
      bAlternateSetting       0
      bNumEndpoints           1
      bInterfaceClass         3 Human Interface Device
      bInterfaceSubClass      0 No Subclass
      bInterfaceProtocol      0 None
      iInterface              0 
        HID Device Descriptor:
          bLength                 9
          bDescriptorType        33
          bcdHID               1.11
          bCountryCode            0 Not supported
          bNumDescriptors         1
          bDescriptorType        34 Report
          wDescriptorLength     203
          Report Descriptor: (length is 203)
            Item(Global): Usage Page, data= [ 0x01 ] 1
                            Generic Desktop Controls
            Item(Local ): Usage, data= [ 0x80 ] 128
                            System Control
            Item(Main  ): Collection, data= [ 0x01 ] 1
                            Application
            Item(Global): Report ID, data= [ 0x01 ] 1
            Item(Local ): Usage Minimum, data= [ 0x81 ] 129
                            System Power Down
            Item(Local ): Usage Maximum, data= [ 0x83 ] 131
                            System Wake Up
            Item(Global): Logical Minimum, data= [ 0x00 ] 0
            Item(Global): Logical Maximum, data= [ 0x01 ] 1
            Item(Global): Report Count, data= [ 0x03 ] 3
            Item(Global): Report Size, data= [ 0x01 ] 1
            Item(Main  ): Input, data= [ 0x02 ] 2
                            Data Variable Absolute No_Wrap Linear
                            Preferred_State No_Null_Position Non_Volatile Bitfield
            Item(Global): Report Count, data= [ 0x01 ] 1
            Item(Global): Report Size, data= [ 0x05 ] 5
            Item(Main  ): Input, data= [ 0x01 ] 1
                            Constant Array Absolute No_Wrap Linear
                            Preferred_State No_Null_Position Non_Volatile Bitfield
            Item(Main  ): End Collection, data=none
            Item(Global): Usage Page, data= [ 0x0c ] 12
                            Consumer
            Item(Local ): Usage, data= [ 0x01 ] 1
                            Consumer Control
            Item(Main  ): Collection, data= [ 0x01 ] 1
                            Application
            Item(Global): Report ID, data= [ 0x02 ] 2
            Item(Global): Logical Minimum, data= [ 0x00 ] 0
            Item(Global): Logical Maximum, data= [ 0x01 ] 1
            Item(Global): Report Count, data= [ 0x12 ] 18
            Item(Global): Report Size, data= [ 0x01 ] 1
            Item(Local ): Usage, data= [ 0x83 0x01 ] 387
                            AL Consumer Control Configuration
            Item(Local ): Usage, data= [ 0x8a 0x01 ] 394
                            AL Email Reader
            Item(Local ): Usage, data= [ 0x92 0x01 ] 402
                            AL Calculator
            Item(Local ): Usage, data= [ 0x94 0x01 ] 404
                            AL Local Machine Browser
            Item(Local ): Usage, data= [ 0xcd ] 205
                            Play/Pause
            Item(Local ): Usage, data= [ 0xb7 ] 183
                            Stop
            Item(Local ): Usage, data= [ 0xb6 ] 182
                            Scan Previous Track
            Item(Local ): Usage, data= [ 0xb5 ] 181
                            Scan Next Track
            Item(Local ): Usage, data= [ 0xe2 ] 226
                            Mute
            Item(Local ): Usage, data= [ 0xea ] 234
                            Volume Decrement
            Item(Local ): Usage, data= [ 0xe9 ] 233
                            Volume Increment
            Item(Local ): Usage, data= [ 0x21 0x02 ] 545
                            AC Search
            Item(Local ): Usage, data= [ 0x23 0x02 ] 547
                            AC Home
            Item(Local ): Usage, data= [ 0x24 0x02 ] 548
                            AC Back
            Item(Local ): Usage, data= [ 0x25 0x02 ] 549
                            AC Forward
            Item(Local ): Usage, data= [ 0x26 0x02 ] 550
                            AC Stop
            Item(Local ): Usage, data= [ 0x27 0x02 ] 551
                            AC Refresh
            Item(Local ): Usage, data= [ 0x2a 0x02 ] 554
                            (null)
            Item(Main  ): Input, data= [ 0x02 ] 2
                            Data Variable Absolute No_Wrap Linear
                            Preferred_State No_Null_Position Non_Volatile Bitfield
            Item(Global): Report Count, data= [ 0x01 ] 1
            Item(Global): Report Size, data= [ 0x0e ] 14
            Item(Main  ): Input, data= [ 0x01 ] 1
                            Constant Array Absolute No_Wrap Linear
                            Preferred_State No_Null_Position Non_Volatile Bitfield
            Item(Main  ): End Collection, data=none
            Item(Global): Usage Page, data= [ 0x01 ] 1
                            Generic Desktop Controls
            Item(Local ): Usage, data= [ 0x06 ] 6
                            Keyboard
            Item(Main  ): Collection, data= [ 0x01 ] 1
                            Application
            Item(Global): Report ID, data= [ 0x04 ] 4
            Item(Global): Usage Page, data= [ 0x07 ] 7
                            Keyboard
            Item(Global): Report Count, data= [ 0x01 ] 1
            Item(Global): Report Size, data= [ 0x08 ] 8
            Item(Main  ): Input, data= [ 0x03 ] 3
                            Constant Variable Absolute No_Wrap Linear
                            Preferred_State No_Null_Position Non_Volatile Bitfield
            Item(Global): Report Count, data= [ 0xe8 ] 232
            Item(Global): Report Size, data= [ 0x01 ] 1
            Item(Global): Logical Minimum, data= [ 0x00 ] 0
            Item(Global): Logical Maximum, data= [ 0x01 ] 1
            Item(Global): Usage Page, data= [ 0x07 ] 7
                            Keyboard
            Item(Local ): Usage Minimum, data= [ 0x00 ] 0
                            No Event
            Item(Local ): Usage Maximum, data= [ 0xe7 ] 231
                            GUI Right
            Item(Main  ): Input, data= [ 0x00 ] 0
                            Data Array Absolute No_Wrap Linear
                            Preferred_State No_Null_Position Non_Volatile Bitfield
            Item(Main  ): End Collection, data=none
            Item(Global): Usage Page, data= [ 0x01 ] 1
                            Generic Desktop Controls
            Item(Local ): Usage, data= [ 0x02 ] 2
                            Mouse
            Item(Main  ): Collection, data= [ 0x01 ] 1
                            Application
            Item(Global): Report ID, data= [ 0x05 ] 5
            Item(Local ): Usage, data= [ 0x01 ] 1
                            Pointer
            Item(Main  ): Collection, data= [ 0x00 ] 0
                            Physical
            Item(Global): Usage Page, data= [ 0x09 ] 9
                            Buttons
            Item(Local ): Usage Minimum, data= [ 0x01 ] 1
                            Button 1 (Primary)
            Item(Local ): Usage Maximum, data= [ 0x08 ] 8
                            (null)
            Item(Global): Logical Minimum, data= [ 0x00 ] 0
            Item(Global): Logical Maximum, data= [ 0x01 ] 1
            Item(Global): Report Count, data= [ 0x08 ] 8
            Item(Global): Report Size, data= [ 0x01 ] 1
            Item(Main  ): Input, data= [ 0x02 ] 2
                            Data Variable Absolute No_Wrap Linear
                            Preferred_State No_Null_Position Non_Volatile Bitfield
            Item(Global): Usage Page, data= [ 0x01 ] 1
                            Generic Desktop Controls
            Item(Local ): Usage, data= [ 0x30 ] 48
                            Direction-X
            Item(Local ): Usage, data= [ 0x31 ] 49
                            Direction-Y
            Item(Global): Logical Minimum, data= [ 0x01 0xf8 ] 63489
            Item(Global): Logical Maximum, data= [ 0xff 0x07 ] 2047
            Item(Global): Report Count, data= [ 0x02 ] 2
            Item(Global): Report Size, data= [ 0x0c ] 12
            Item(Main  ): Input, data= [ 0x06 ] 6
                            Data Variable Relative No_Wrap Linear
                            Preferred_State No_Null_Position Non_Volatile Bitfield
            Item(Local ): Usage, data= [ 0x38 ] 56
                            Wheel
            Item(Global): Logical Minimum, data= [ 0x81 ] 129
            Item(Global): Logical Maximum, data= [ 0x7f ] 127
            Item(Global): Report Count, data= [ 0x01 ] 1
            Item(Global): Report Size, data= [ 0x08 ] 8
            Item(Main  ): Input, data= [ 0x06 ] 6
                            Data Variable Relative No_Wrap Linear
                            Preferred_State No_Null_Position Non_Volatile Bitfield
            Item(Global): Usage Page, data= [ 0x0c ] 12
                            Consumer
            Item(Local ): Usage, data= [ 0x38 0x02 ] 568
                            AC Pan
            Item(Global): Report Count, data= [ 0x01 ] 1
            Item(Main  ): Input, data= [ 0x06 ] 6
                            Data Variable Relative No_Wrap Linear
                            Preferred_State No_Null_Position Non_Volatile Bitfield
            Item(Main  ): End Collection, data=none
            Item(Main  ): End Collection, data=none
      Endpoint Descriptor:
        bLength                 7
        bDescriptorType         5
        bEndpointAddress     0x82  EP 2 IN
        bmAttributes            3
          Transfer Type            Interrupt
          Synch Type               None
          Usage Type               Data
        wMaxPacketSize     0x0040  1x 64 bytes
        bInterval               1
Device Status:     0x0000
  (Bus Powered)
root@cerberus:/home/jlquinn# 
```

The first interface is the standard USB HID keyboard interface.

Driver writes commands to 2nd interface OUT port.  IN port sends
acknowledgement of commands.

I *think* that the 3rd interface supports the keyboard generating mouse
keyclicks and some media keys.


# Commands from Host to XBows

I've found that what comes back from the keyboard seems pretty uninteresting,
so all of the following is packets sent from the driver to the keyboard.

Commands are 64 bytes long.  First 6 bytes are command.  Bytes 8-63 are data
payload.  Bytes 6,7 are CRC-16/CCiTT-FALSE, computed with bytes 6,7 set to 0
first.  Then they get replaced with the CRC 16 bit value in little-endian
format.

It appears that the CRC calculation is CRC-16/CCITT-FALSE computed by setting
the crc field to 00 00, then inserting the values swapped.  For example:
1a0201000000a08000000000000000000000000000000000... replacing a080 with 0000
and computing CRC-16/CCiTT-FALSE = 80a0

```
This works on another packet too:
1a 01 88 01 00 38 aa 4b 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 ff 00 cb 64 ff 00 cb 64 00 00 00 00
```

Generally, the driver keeps sending
0c0000000000a70d00000000000000000000000000000000... when there's nothing to
do.  It does this about every 0.3 seconds.

# Driver Mode Programming

I've made progress on the how the driver controls lights in the driver layer.
The driver layer only does anything when the host actively controls it.

Starting the windows driver program kicks the keyboard into driver mode.  I'm
not sure yet what the command is that really does this.

The driver appears to send 3 lighting sequences followed by the 0c00 idle.  A
lighting sequence looks like:

```
1a0100000038 crc ...
1a0138000038 crc ...
1a0170000038 crc ...
1a01a8000038 crc ...
1a01e0000038 crc ...
1a0118010038 crc ...
1a0150010038 crc ...
1a0188010038 crc ...
1a01c0010038 crc ...
1a01f8010038 crc ...
1a0200000038 crc ...    End of Sequence
```

The first 10 commands are lighting data per key. The 11th I assume is an end
of sequence command.

Within a command, the 0x38 refers to how many data bytes are valid.

Each of the 1a01 commands is followed by 2 bytes indicating byte counts, I
think.  3800 is little endian representation of 0x0038.  The next command is
0x0070, which is 0x38 more than 0x38.  Each successive command is 0x38 larger
in bytes 2,3 interpreted as little endian 16 bit int.

For a static light pattern, all the sequences would be the same.  Dynamic
patterns would alter the data in the light sequence to animate things.

Within the data section, each group of 4 bytes seems to be:
```
RRGGBB64
```
The 0x64 seems to indicate that the key LED has been programmed, rather than
in default state.

## Static light setting and mapping to key LEDs.

Here, each key label or ?? is a group of 4 bytes as above.  ?? indicates I
don't know what this maps to.  Some may refer to keypad keys.  Some probably
refer to the standard PC keys not present on the XBows keyboard, such as
Scroll Lock.  Some likely refer to the various media control and other kinds
of keys.  Take a look at Functions in the windows driver, and look through the
Key base function set choices.  I counted about 140 in total.

```
1a 01 00 00 00 38 CRC16  Esc		F1
F2			F3     	  	 F4			F5
F6			??		 	 F7  		F8
F9			F10    		 F11	 	F12

1a 01 38 00 00 38 CRC16  Del		??
PrtSc 		??     	  	 ??			??
??			??		 	 `~			1!
2@			??		 	 3#			4$

1a 01 70 00 00 38 c4 a6  5%			??
6^    	 	7&    	     8*			??
9(	 		0)		 	 -_			=+
bksp	 	??		 	 ??			??

1a 01 a8 00 00 38 b7 35	 ??			??
tab   	 	Q     	     W			??
E	 		R		 	 T			XBOW
Y	 		U		 	 I			??

1a 01 e0 00 00 38 c1 42	 O			P
[{    	 	]}    	     \|			pgup
??	 		??		 	 ??			??
caps	 	A		 	 S			??

1a 01 18 01 00 38 b3 86	 D			F
G     	 ctrbksp     	 H			J
K			??		 	 L			;:
'"	 		enter		 ??			pgdn

1a 01 50 01 00 38 c6 d1	 ??			??
??    	 ??    	     	 Lshift		Z
X	 	 ??		 		 C			V
B	 ctrenter	 		 N			M

1a 01 88 01 00 38 d5 82	 ,<			??
.>    	 	/?    	     Rshift		??
up	 		??			??			??
??	 		??			Lctrl		win	

1a 01 c0 01 00 38 d2 c8	 Lalt		??
??    	 	Lspc  	     Mctrl		??
Mshift	 	Rspc		 Ralt		??
??	 		Fn		 	Rctrl		left

1a 01 f8 01 00 18 45 ec	 down		right
??    	 ??    	     	 ??		??
??    	 ??    	     	 ??		??
??    	 ??    	     	 ??		??

1a 20 00 00 00 00 crc	 ??		??
```


I think this is enough to program light patterns in the driver layer.

## Modifying Key Mappings in Driver Mode

I think that we have a sequence that looks like this to set the key bindings
in driver mode.

```
0b0500000000 crc 00000000 x 14
010900000000 crc 00000000 x 14
160100000038 crc ffffffff ffffffff
				 01000002 02000002 04000002 08000002	Ls Lc La Win
	     		 10000002 20000002 40000002 ffffffff	Rc Rs Ra
				 00040002 00050002 00060002 00070002	A B C D
160138000038 crc 00080002 00090002						E F
				 000a0002 000b0002 000c0002 000d0002	G H I J
	     		 000e0002 000f0002 00100002 00110002	K L M N
				 00120002 00130002 00140002 00150002	O P Q R
160170000038 crc 00160002 00170002						S T
				 00180002 00190002 001a0002 001b0002	U V W X
	     		 001c0002 001d0002 001e0002 001f0002	Y Z 1 2
				 00200002 00210002 00220002 00230002	3 4 5 6
1601a8000038 crc 00240002 00250002						7 8
				 00260002 00270002 00280002 00290002	9 0 Renter Esc
				 002a0002 002b0002 002c0002 002d0002	BS Tab Lspc -_
				 002e0002 002f0002 00300002 00310002	=+ [{ ]} \|
1601e0000038 crc 00330002 00340002						;: '"
				 00350002 00360002 00370002 00380002	`~ ,< .> /?
				 00390002 003a0002 003b0002 003c0002	Caps F1 F2 F3
				 003d0002 003e0002 003f0002 00400002	F4 F5 F6 F7
160118010038 crc 00410002 00420002						F8 F9
				 00430002 00440002 00450002 00460002	F10 F11 F12 PrtScrn
	     		 ffffffff x 4
				 004b0002 004c0002 ffffffff 004e0002	PgU Del ?? PgD
160150010038 crc 004f0002 00500002						Rt Lt
				 00510002 00520002						Dn Up
				 ffffffff x 10
160188010038 crc ffffffff x 14
1601c0010020 crc 002a0002 00280002						Mbs Menter
				 01000002 20000002 002c0002				Mc  Ms     Rspc
	     		 ffffffff x 3
				 00000000 x 6
160300000038 crc ffffffff x 14
160338000038 crc ffffffff x 14
160370000008 crc ffffffff ffffffff 00000000 x 12
170100000000 crc f0011203 00000000 0f000000 00000000 x 11
0c0000000000 crc end of sequence code
```

Here, the first 2 commands 0b05 and 0109 appear to the start of the remapping
commands.

The data commands show the keys they correspond to.  Remapping involves
putting a different key's code in that spot.  For example, to remap Esc to Q,
the 00290002 value would be changed to 00140002.

I assume that the various unlabeled ints are just part of the programming.
Some might be the numeric keypad but I don't have one, so I can't test that
theory for now.

Commands 1603 and 1701 appear to just terminate the programming sequence.


# Custom Layer Programming

This section contains notes on how programming the XBows custom layers works.

Here is a trace of the driver programming the lights in a custom layer.

```
010900000000 crc 00
210301000000 crc 00
220300003800 crc ffffffff ffffffff
			 	 01000002 02000002 04000002 08000002
	     	 	 10000002 20000002 00630002 ffffffff
			 	 00040002 00050002 00060002 00070002 pos 8 different
220338003800 crc 00080002 00090002
	     	 	 000a0002 000b0002 00600002 005c0002	pos 4, 5, 6, 7, 8 10,11 different
			 	 005d0002 005e0002 00590002 00110002
			 	 00610002 00560002 00140002 00150002	different than CUST1
			 	 THIS MAY BE THE KEYPAD OVERLAY!!!!
220370003800 crc 00160002 00170002
	     	 	 005f0002 00190002 001a0002 001b0002    pos 2 is different
			 	 001c0002 001d0002 001e0002 001f0002
			 	 ... 00230002
2203a8003800 crc 00240002 00540002
	     	 	 00550002 00270002 thru 00310002	pos 1,2 different
2203e0003800 crc 00570002 00340002 			pos 0,3,4 different
	     	 00350002 005a0002 005b0002 00580002
			 00390002 thru 00400002
220318013800 crc 00410002 ... 00460002 (6 ints)
	     	 ffff (4 ints) these line up with missing expected ints
			 004b0002 004c0002 ffff 004e0002  	  SAME AS 160118...
220350013800 crc same as 160150010038
220388013800 crc ff x 14 same as 160188010038
2203c0012000 crc 002a0002 00280002 01000002 20000002 00620002
	     	 ffffffff (3 ints)
			 00 (6 ints)  NOT same as 1601c0010020  pos 4 different
	     	 lighting program starts here?
210304000000 crc 0000 (14 ints)
210305000000 crc 0000 (14 ints)
260300003800 crc ffff (14 ints)
260338003800 crc ffff (14 ints)
260370000800 crc ffff ffff 00 (12 ints)
210306000000 crc 00 x 14
270300000038 crc 00020000 03000000
	     	 4e020000 01000000 ff x 10
270338000038 crc ff x 14
270370000038 crc ff x 14
2703a8000038 crc ff x 14
2703e0000038 crc ff x 14	packet 467
270318010038 crc ff x 14
270350010038 crc ff x 14
270388010038 crc ff x 14
2703c0010038 crc ff x 14
2703f8010038 crc ff ff
	     	 03001600 ff x 3
			 ffffffff 0f000000 00000300 1600ffff
			 ff x 3   	   	    ffff0f00
270330020038 crc 00000000 03001600
	     	 ff x 4
			 0f000000 00000020 ff x 2
			 ff x 2	  	   0f000000 0000ff00
270368020006 crc 00000000 79000000
	     	 00 x 12
0b0300000000 crc 00 x 14
0c0000000000 	      at 3.0737sec
```

The programming sequence starts with the 0109 command.  This shows up in the
driver layer programming as well, but here it's the first packet.

Remaining packets encode the custom layer being programmed in byte 1.  For
example:

`
270368020006
`

Here, byte 1 is 0x03.  0x02 addresses custom layer 1, 0x03 is layer 2, and
0x04 is layer 3.

Command 2 is 0x21.  This is followed by a bunch of 0x22 commands.  The 0x22
commands affect the keyboard mapping for the custom layer.

After 9 packets of 0x22 commands, there is a 6 packet sequence of 0x21 0x21
0x26 0x26 0x26 0x21.  I don't know what this does other than separate keyboard
from lighting.

Programming the lights happens with 0x27 commands.  This sequence is variable
length.

The sequence is terminated with the 0x0b command.

Question: if the keyboard were programmed with 2201 commands, would this
permanently write to the driver layer?

## Custom Layer Keyboard Programming

As mentioned above, the sequence of 0x22 commands remaps the keyboard of a
custom layer.

Bytes 2 and 3 form a 16 bit little-endian int representing the index of the
data bytes give so far (I think).

Byte 4 gives the number of valid data bytes.  Most packets will have 0x38
here.  The last one may be less.

I *think* that the 0x22 command sequence is the same as the 0x16 command
sequence that programs the keymap in the driver layer.  I don't know for sure
yet.


TODO: I still haven't explored how macros are programmed.  I haven't explored
the Fnx key.




## Custom Layer Light Programming

The DIY Lights section of the driver enables writing new patterns.  The driver
explains a few things to us.


The driver assembles patterns from two building blocks: animation frames, and
lighting frames.  

Animation frames consist of a duration and a set of active keys.

Light frames are a little more complex.  They have a duration and a set of
active keys.  Lights can have one of three patterns: monochrome, RGB cycle,
and breathing.  The light pattern also has a starting color.

These features appear to show up in the USB programming model as well.

One piece of information is that a lighting frame applies to a set of active
keys using a bitmap.  The bits are 1 if the key is active, 0 otherwise.

Using the Calculator light program, I get a total sequence as follows:

```
010900000000 crc  00000000 x 14

210301000000 crc  00000000 x 14

220300003800 crc  ffffffff ffffffff
01000002 02000002 04000002 08000002
10000002 20000002 00630002 ffffffff
00040002 00050002 00060002 00070002

220338003800 crc  00080002 00090002
000a0002 000b0002 00600002 005c0002
005d0002 005e0002 00590002 00110002
00610002 00560002 00140002 00150002

220370003800 crc  00160002 00170002
005f0002 00190002 001a0002 001b0002
001c0002 001d0002 001e0002 001f0002
00200002 00210002 00220002 00230002

2203a8003800 crc  00240002 00540002
00550002 00270002 00280002 00290002
002a0002 002b0002 002c0002 002d0002
002e0002 002f0002 00300002 00310002

2203e0003800 crc  00570002 00340002
00350002 005a0002 005b0002 00580002
00390002 003a0002 003b0002 003c0002
003d0002 003e0002 003f0002 00400002

220318013800 crc  00410002 00420002
00430002 00440002 00450002 00460002
ffffffff ffffffff ffffffff ffffffff
004b0002 004c0002 ffffffff 004e0002

220350013800 crc  004f0002 00500002
00510002 00520002 ffffffff ffffffff
ffffffff ffffffff ffffffff ffffffff
ffffffff ffffffff ffffffff ffffffff

220388013800 crc  ffffffff ffffffff
ffffffff ffffffff ffffffff ffffffff
ffffffff ffffffff ffffffff ffffffff
ffffffff ffffffff ffffffff ffffffff

2203c0012000 crc  002a0002 00280002
01000002 20000002 00620002 ffffffff
ffffffff ffffffff 00000000 00000000
00000000 00000000 00000000 00000000

210304000000 crc  00000000 00000000
00000000 00000000 00000000 00000000
00000000 00000000 00000000 00000000
00000000 00000000 00000000 00000000

210305000000 crc  00000000 00000000
00000000 00000000 00000000 00000000
00000000 00000000 00000000 00000000
00000000 00000000 00000000 00000000

260300003800 crc  ffffffff ffffffff
ffffffff ffffffff ffffffff ffffffff
ffffffff ffffffff ffffffff ffffffff
ffffffff ffffffff ffffffff ffffffff

260338003800 crc  ffffffff ffffffff
ffffffff ffffffff ffffffff ffffffff
ffffffff ffffffff ffffffff ffffffff
ffffffff ffffffff ffffffff ffffffff

260370000800 crc  ffffffff ffffffff
00000000 00000000 00000000 00000000
00000000 00000000 00000000 00000000
00000000 00000000 00000000 00000000

210306000000 crc  00000000 00000000
00000000 00000000 00000000 00000000
00000000 00000000 00000000 00000000
00000000 00000000 00000000 00000000

270300000038 crc  00020000 01000000		start of lighting program, 01000000
1a020000 01000000 ffffffff ffffffff		seems to be total animation frame count
ffffffff ffffffff ffffffff ffffffff
ffffffff ffffffff ffffffff ffffffff

270338000038 crc  ffffffff ffffffff
ffffffff ffffffff ffffffff ffffffff
ffffffff ffffffff ffffffff ffffffff
ffffffff ffffffff ffffffff ffffffff

270370000038 crc  ffffffff ffffffff
ffffffff ffffffff ffffffff ffffffff
ffffffff ffffffff ffffffff ffffffff
ffffffff ffffffff ffffffff ffffffff

2703a8000038 crc  ffffffff ffffffff
ffffffff ffffffff ffffffff ffffffff
ffffffff ffffffff ffffffff ffffffff
ffffffff ffffffff ffffffff ffffffff

2703e0000038 crc  ffffffff ffffffff
ffffffff ffffffff ffffffff ffffffff
ffffffff ffffffff ffffffff ffffffff
ffffffff ffffffff ffffffff ffffffff

270318010038 crc  ffffffff ffffffff
ffffffff ffffffff ffffffff ffffffff
ffffffff ffffffff ffffffff ffffffff
ffffffff ffffffff ffffffff ffffffff

270350010038 crc  ffffffff ffffffff
ffffffff ffffffff ffffffff ffffffff
ffffffff ffffffff ffffffff ffffffff
ffffffff ffffffff ffffffff ffffffff

270388010038 crc  ffffffff ffffffff
ffffffff ffffffff ffffffff ffffffff
ffffffff ffffffff ffffffff ffffffff
ffffffff ffffffff ffffffff ffffffff

2703c0010038 crc  ffffffff ffffffff
ffffffff ffffffff ffffffff ffffffff
ffffffff ffffffff ffffffff ffffffff
ffffffff ffffffff ffffffff ffffffff

2703f8010038 crc  ffffffff ffffffff
03001600 00000000 05006003 00d80000
36008001 00000000 00000020 00000000
05006003 00d80000 36008001 00000000

27033002000a crc  000000ff 00000000
79000000 00000000 00000000 00000000
00000000 00000000 00000000 00000000
00000000 00000000 00000000 00000000

0b0300000000 crc  00000000 00000000
00000000 00000000 00000000 00000000
00000000 00000000 00000000 00000000
00000000 00000000 00000000 00000000
```

0x0109 and 0x2103 seem to always start the custom layer program.  Byte 1 of
0x21 is the layer code 03 for custom layer 2.

Lighting seems to start at command 0x27.  Based on some tests, 

At 2703f801 I believe we have a bitmap of keys with lights enabled for the
pattern.  The specific command number doesn't appear to be fixed.  This is
just 0x01f8 bytes after the start.

03001600 appears to be a prefix for the light frame key bitmap.  I think the
0xffffffff ffffffff just has to be there as well.

I copied and modified the Calculator pattern to enable a single light at a
time to try to describe the bitmap.  Treating the first byte after the crc as
byte 0:

```
byte 0 bits 0-7		Esc  F1   F2   F3   F4   F5   F6   ??
byte 1 bits 0-7		F7   F8   F9   F10  F11  F12  Del  ??
byte 2 bits 0-7     Prt  ??   ??   ??   ??   ??   ~    1
byte 3 bits 0-7     2    ??   3    4    5    ??   6    7
byte 4 bits 0-7     8    ??   9    0    -    =    BkSp ??
byte 5 bits 0-7     ??   ??   ??   ??   Tab  Q    W    ??
byte 6 bits 0-7     E    R    T    Xbow Y    U    I    ??
byte 7 bits 0-7     O    P    [    ]    \    PgUp ??   ??
byte 8 bits 0-7     ??   ??   Caps A    S    ??   D    ??
byte 9 bits 0-7     ??   ??   ??   ??   K    ??   L    ;
byte 10 bits 0-7    '    Entr ??   PgDn ??   ??   ??   ??
byte 11 bits 0-7    LShf Z    X    ??   C    V    B    Entr
byte 12 bits 0-7    N    M    ,    ??   .    /    RShf ??
byte 13 bits 0-7    Up   ??   ??   ??   ??   ??   LCtl Win
byte 14 bits 0-7    LAlt ??   ??   LSpc MCtl ??   MShf RSpc
byte 15 bits 0-7    RAlt ??   ??   Fn   RCtl Left Down Rght
```

This is 16 bytes, 4 ints.  After this comes 0x00000000 0x00000020.  I assume
this is a required separator.  The first int might be more keymap bits though.

Command 27033002000a contains 10 bytes that are valid (see the 0x0a at the end of
the command).

000000ff 00000000 7900

The 0x79 near the end always seems to show up.  The ff is the green level.  So
in reality we have:

0000RRGG BB000000 7900

I don't yet know what indicates monochrome, RGB cycle, or breathing.


### Experiment on lighting

I'm trying this on custom layer 2.

I have 2 frames of keyboard animation:

	Frame 1 Esc
	Frame 2 F1
	
Also 3 frames of lights.

	Frame 1 is Esc, FF00DD (pink), monochrome.
	Frame 2 is F1, 00FFEE (cyan), RBG cycle.
	Frame 3 is All keys, BBAADD (grayish), Breathing.
	
Differing frame counts and colors will hopefully let me see various packet
distinctions.

I'm including only the packets starting with 0x27.  The earlier packets appear
unaffected by lighting changes.

```
270300000038 crc  00020000 02000000		start of lighting program, 01000000
34020000 03000000 ffffffff ffffffff		seems to be total animation frame count
ffffffff ffffffff ffffffff ffffffff
ffffffff ffffffff ffffffff ffffffff

270338000038 crc  ffffffff ffffffff		never seems to change
ffffffff ffffffff ffffffff ffffffff
ffffffff ffffffff ffffffff ffffffff
ffffffff ffffffff ffffffff ffffffff

270370000038 crc  ffffffff ffffffff		never seems to change
ffffffff ffffffff ffffffff ffffffff
ffffffff ffffffff ffffffff ffffffff
ffffffff ffffffff ffffffff ffffffff

2703a8000038 crc  ffffffff ffffffff		never seems to change
ffffffff ffffffff ffffffff ffffffff
ffffffff ffffffff ffffffff ffffffff
ffffffff ffffffff ffffffff ffffffff

2703e0000038 crc  ffffffff ffffffff		never seems to change
ffffffff ffffffff ffffffff ffffffff
ffffffff ffffffff ffffffff ffffffff
ffffffff ffffffff ffffffff ffffffff

270318010038 crc  ffffffff ffffffff		never seems to change
ffffffff ffffffff ffffffff ffffffff
ffffffff ffffffff ffffffff ffffffff
ffffffff ffffffff ffffffff ffffffff

270350010038 crc  ffffffff ffffffff		never seems to change
ffffffff ffffffff ffffffff ffffffff
ffffffff ffffffff ffffffff ffffffff
ffffffff ffffffff ffffffff ffffffff

270388010038 crc  ffffffff ffffffff		never seems to change
ffffffff ffffffff ffffffff ffffffff
ffffffff ffffffff ffffffff ffffffff
ffffffff ffffffff ffffffff ffffffff

2703c0010038 crc  ffffffff ffffffff		never seems to change
ffffffff ffffffff ffffffff ffffffff
ffffffff ffffffff ffffffff ffffffff
ffffffff ffffffff ffffffff ffffffff

2703f8010038 crc  ffffffff ffffffff
03001600 01000000 00000000 00000000
00000000 00000000 00000300 16000200
00000000 00000000 00000000 00000000

27033002000a crc  00000000 00200100
00000000 00000000 00000000 00000000
00000000 ff00dd00 00007900 00200200
00000000 00000000 00000000 00000000

27036802002c crc  00000000 00ffee00
00007900 00207f7f c1dd7d70 7f3fdcdf
0bf777c1 d9f90000 00000000 bbaadd00
00007900 00000000 00000000 00000000

0b0300000000 crc  00000000 00000000
00000000 00000000 00000000 00000000
00000000 00000000 00000000 00000000
00000000 00000000 00000000 00000000
```

Packet 270300000038 by ints

	00020000		start of lighting program?
	02000000		Indicates 2 frames of animation
	34020000		The 0x34 is 2 * 0x1a.  Seems matched to animation frame count
	03000000		Indicates 3 frames of lighting
	ffffffff		Remainder of packet.  I haven't seen it change.

Packet 2703f8010038 by int

	ffffffff		2 fixed ints
	03001600		Start of animation key bitmap for 22 bytes
	01000000		Esc is bit 0, byte 0.
	00000000 00000000 00000000 00000000		No other keys set
	00000300		2 bytes of 0.  Then start of 03001600 (key bitmap start)
	16000200		2 bytes of 03001600.  F1 is byte 0, bit 1
	00000000 00000000 00000000 00000000		No keys set

Packet 27033002000a by int

	00000000		Last 4 bytes of key bitmap
	00200100		I think this is 0020 as start of lighting frame.  Then 0100
					is 2 bytes of lighting keymap with Esc set 
	00000000 00000000 00000000 00000000		16 bytes of keymap
	00000000		last 4 bytes of lighting key bitmap
	ff00dd00		RRGGBB00 is color for this frame
	00007900		lighting frame terminator
	01200200		0120 start of lighting frame #2.  01 indicating RGB cycle,
					Then 0200 is F1 in keymap
	00000000 00000000 00000000 00000000		16 bytes of keymap

Packet 27036802002c by int

	00000000		last 4 bytes of keymap
	00ffee00		RRGGBB00 color for this frame
	78007900		lighting frame terminator.  Is 78 related to frame length?
	02207f7f		0220 start of lighting frame,  02 is breathing.
					First 2 bytes of keymap is 7f7f
	c1dd7d70 7f3fdcdf 0bf777c1 d9f90000		16 bytes of keymap
	00000000		4 bytes of keymap
	bbaadd00		RRGGBB00
	10000500		frame terminator? 
	00000000 00000000 00000000		unused bytes.  0x2c at end of command
									ignores them

Monochrome frames appear to terminate with 00007900.  I'm seeing an RGB cycle
frame end in 78007900.  Breathing frame ending in 10000500.  However, the
leading byte of the frame also has a different code. 

The monochrome frame is duration 2.  The RGB frame is duration 3.  The
breathing frame is 6+5.

I don't understand the frame terminator.

Dropping the breathing frame, I have a monochrome frame duration 4, RGB frame
duration 3.  I repeat the capture with monochrome frame duration 7, RGB frame
duration 2.

Packet 27026802000c		duration 4/3

	78 00 79 00			01111000 00 79 00
	
Packet 27026802000c		duration 7/2

	b4 00 79 00			10110100 00 79 00
	
Clues but no answer yet.

### Simplifying things

1 frame animation, duration 20, Esc selected.
1 frame lighting, duration 1, Esc selected, Red FF0000, monochrome

```
270300000038 crc  00020000 14000000		start of lighting program, 01000000
08040000 01000000 ffffffff ffffffff		seems to be total animation frame count
ffffffff ffffffff ffffffff ffffffff
ffffffff ffffffff ffffffff ffffffff
```

0x14 is 20.  This is the animation frame duration.

Now 0804.  I normally see 1a02 when the animation is duration 1.  Here, if I
first subtract 02 from 04, I have 0802.  Treating this as a little endian
short, I have 0x0208.  0x0208 / 20 gives 0x1a, which is the number I see in
duration 1 animations.

So this suggests that those 2 bytes are 0x0200 + 0x1a * duration.

I next see 20 copies of 03001600 followed by a 22 byte keymap.  After that is:

	0020	 followed by 22 byte keymap
	RRGGBB00
	00007900
	
Now let's reverse this.  1 duration animation, 20 duration lighting.

```
270300000038 crc  00020000 01000000		start of lighting program 1 anim frame
1a020000 01000000 ffffffff ffffffff		There's 01 here, where I was expecting 0x14
ffffffff ffffffff ffffffff ffffffff
ffffffff ffffffff ffffffff ffffffff
```

I can't see the effect.  Changing to 5 duration lighting has no difference with
20 duration lighting.  I think the animation duration has to exceed the lighting
duration.

Trying duration 20 animation, duration 20.  This is no different than duration
20 animation, duration 1 lighting.  Perhaps the driver just figures that
there's no need to do anything different.

### Trying RGB cycle durations

Windmill has 1 frame animation, 11 frames of lighting, each duration 60.  I
tried changing the durations from 60 down to 10 by 5's.  

```
270300000038 crc  00020000 01000000		start of lighting program 1 anim frame
1a020000 0b000000 ffffffff ffffffff		The 0x0b seems to number of light frames
ffffffff ffffffff ffffffff ffffffff
ffffffff ffffffff ffffffff ffffffff
```

0x0b is 11, which is the number of light frames, not the durations.

In the lighting frames, I see the RGB0 followed by a number before the 0x79
frame terminator.

The 11 sequences are:

0120 keymap RRGGBB00 06007900
0120 keymap RRGGBB00 06007900
0120 keymap RRGGBB00 07007900
0120 keymap RRGGBB00 08007900
0120 keymap RRGGBB00 09007900
0120 keymap RRGGBB00 0a007900
0120 keymap RRGGBB00 0c007900
0120 keymap RRGGBB00 0e007900
0120 keymap RRGGBB00 12007900
0120 keymap RRGGBB00 18007900
0120 keymap RRGGBB00 24007900

The first byte of the 79 int looks like an inverse of the duration.  We have

	byte  duration
	6		60
	6		55
	7		50
	8		45
	9		40
	10		35
	12		30
	14		25
	18		20
	24		15
	36		10

The duration is the inverse of the byte value.  So x/60 = 6 gives x=360.  This
works for 36,10.  In general, it looks like:

	byte = floor(360/duration)
	