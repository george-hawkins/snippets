Bluetooth
=========

There are an excess of commands that can you show you some information or other about Bluetooth on your Linux system.

This page shows the output of all of them along with pointers to how to manage the device (e.g. to pair it with some other Bluetooth device).

See the ArchLinux Bluetooth [wiki page](https://wiki.archlinux.org/title/bluetooth) for much more information and a proper description of pairing etc.

---

`dmesg` shows the kernel messages output, you can see the Bluetooth related output that occurred during startup:

```
$ sudo dmesg | fgrep -i -e blue -e btusb
[    3.678982] Bluetooth: Core ver 2.22
[    3.679434] NET: Registered PF_BLUETOOTH protocol family
[    3.679436] Bluetooth: HCI device and connection manager initialized
[    3.679438] Bluetooth: HCI socket layer initialized
[    3.679440] Bluetooth: L2CAP socket layer initialized
[    3.679444] Bluetooth: SCO socket layer initialized
[    3.742751] usbcore: registered new interface driver btusb
[    3.742826] Bluetooth: hci0: Firmware revision 0.0 build 121 week 36 2020
[    3.808806] Bluetooth: hci0: MSFT filter_enable is already on
                          ^^^^
[    4.292510] Bluetooth: BNEP (Ethernet Emulation) ver 1.3
[    4.292513] Bluetooth: BNEP filters: protocol multicast
[    4.292515] Bluetooth: BNEP socket layer initialized
[    6.037779] Bluetooth: RFCOMM TTY layer initialized
[    6.037788] Bluetooth: RFCOMM socket layer initialized
[    6.037791] Bluetooth: RFCOMM ver 1.11
```

`hci0` is the Bluetooth interface - you'll find it exposed via `/sys/class/bluetooth`:

```
$ ls /sys/class/bluetooth
hci0@
$ readlink -f /sys/class/bluetooth/hci0
/sys/devices/pci0000:00/0000:00:14.0/usb1/1-14/1-14:1.0/bluetooth/hci0
$ ls /sys/class/bluetooth/hci0/
device@  power/  rfkill0/  subsystem@  uevent
```

`hci0` is just a softlink to the actual USB device but there's nothing very human consumable to see here.

---

You can get the same output, but with human readable timestamps, using `journalctl`:

```
$ journalctl --boot --dmesg | fgrep -i blue
Apr 03 21:05:00 example-host kernel: Bluetooth: Core ver 2.22
Apr 03 21:05:00 example-host kernel: NET: Registered PF_BLUETOOTH protocol family
Apr 03 21:05:00 example-host kernel: Bluetooth: HCI device and connection manager initialized
Apr 03 21:05:00 example-host kernel: Bluetooth: HCI socket layer initialized
Apr 03 21:05:00 example-host kernel: Bluetooth: L2CAP socket layer initialized
Apr 03 21:05:00 example-host kernel: Bluetooth: SCO socket layer initialized
Apr 03 21:05:00 example-host kernel: Bluetooth: hci0: Firmware revision 0.0 build 121 week 36 2020
Apr 03 21:05:00 example-host kernel: Bluetooth: hci0: MSFT filter_enable is already on
Apr 03 21:05:01 example-host kernel: Bluetooth: BNEP (Ethernet Emulation) ver 1.3
Apr 03 21:05:01 example-host kernel: Bluetooth: BNEP filters: protocol multicast
Apr 03 21:05:01 example-host kernel: Bluetooth: BNEP socket layer initialized
Apr 03 21:05:02 example-host kernel: Bluetooth: RFCOMM TTY layer initialized
Apr 03 21:05:02 example-host kernel: Bluetooth: RFCOMM socket layer initialized
Apr 03 21:05:02 example-host kernel: Bluetooth: RFCOMM ver 1.11
```

`journalctl` can also show the output from the SystemD `bluetooth` unit:

```
$ journalctl --boot --unit=bluetooth
Apr 03 21:05:01 example-host systemd[1]: Starting Bluetooth service...
Apr 03 21:05:01 example-host bluetoothd[866]: Bluetooth daemon 5.64
Apr 03 21:05:01 example-host bluetoothd[866]: Starting SDP server
Apr 03 21:05:01 example-host bluetoothd[866]: Bluetooth management interface 1.21 initialized
Apr 03 21:05:01 example-host systemd[1]: Started Bluetooth service.
Apr 03 21:05:02 example-host bluetoothd[866]: Endpoint registered: sender=:1.36 path=/MediaEndpoint/A2DPSink/sbc
Apr 03 21:05:02 example-host bluetoothd[866]: Endpoint registered: sender=:1.36 path=/MediaEndpoint/A2DPSource/sbc
Apr 03 21:05:02 example-host bluetoothd[866]: Endpoint registered: sender=:1.36 path=/MediaEndpoint/A2DPSink/sbc_xq_453
Apr 03 21:05:02 example-host bluetoothd[866]: Endpoint registered: sender=:1.36 path=/MediaEndpoint/A2DPSource/sbc_xq_453
Apr 03 21:05:02 example-host bluetoothd[866]: Endpoint registered: sender=:1.36 path=/MediaEndpoint/A2DPSink/sbc_xq_512
Apr 03 21:05:02 example-host bluetoothd[866]: Endpoint registered: sender=:1.36 path=/MediaEndpoint/A2DPSource/sbc_xq_512
Apr 03 21:05:02 example-host bluetoothd[866]: Endpoint registered: sender=:1.36 path=/MediaEndpoint/A2DPSink/sbc_xq_552
Apr 03 21:05:02 example-host bluetoothd[866]: Endpoint registered: sender=:1.36 path=/MediaEndpoint/A2DPSource/sbc_xq_552
Apr 04 19:55:33 example-host bluetoothd[866]: Controller resume with wake event 0x0
```

You'll also see the last few lines of this output if you ask `systemctl` for the unit's status:

```
● bluetooth.service - Bluetooth service
     Loaded: loaded (/lib/systemd/system/bluetooth.service; enabled; vendor preset: enabled)
     Active: active (running) since Wed 2024-04-03 21:05:01 CEST; 23h ago
       Docs: man:bluetoothd(8)
   Main PID: 866 (bluetoothd)
     Status: "Running"
      Tasks: 1 (limit: 38259)
     Memory: 2.5M
        CPU: 59ms
     CGroup: /system.slice/bluetooth.service
             └─866 /usr/lib/bluetooth/bluetoothd

Apr 03 21:05:01 example-host systemd[1]: Started Bluetooth service.
Apr 03 21:05:02 example-host bluetoothd[866]: Endpoint registered: sender=:1.36 path=/MediaEndpoint/A2DPSink/sbc
Apr 03 21:05:02 example-host bluetoothd[866]: Endpoint registered: sender=:1.36 path=/MediaEndpoint/A2DPSource/sbc
Apr 03 21:05:02 example-host bluetoothd[866]: Endpoint registered: sender=:1.36 path=/MediaEndpoint/A2DPSink/sbc_xq_453
Apr 03 21:05:02 example-host bluetoothd[866]: Endpoint registered: sender=:1.36 path=/MediaEndpoint/A2DPSource/sbc_xq_453
Apr 03 21:05:02 example-host bluetoothd[866]: Endpoint registered: sender=:1.36 path=/MediaEndpoint/A2DPSink/sbc_xq_512
Apr 03 21:05:02 example-host bluetoothd[866]: Endpoint registered: sender=:1.36 path=/MediaEndpoint/A2DPSource/sbc_xq_512
Apr 03 21:05:02 example-host bluetoothd[866]: Endpoint registered: sender=:1.36 path=/MediaEndpoint/A2DPSink/sbc_xq_552
Apr 03 21:05:02 example-host bluetoothd[866]: Endpoint registered: sender=:1.36 path=/MediaEndpoint/A2DPSource/sbc_xq_552
Apr 04 19:55:33 example-host bluetoothd[866]: Controller resume with wake event 0x0
```

---

Even if your Bluetooth device is builtin, there's a good chance it still shows up as a USB device.

```
$ lsusb
Bus 002 Device 001: ID 1d6b:0003 Linux Foundation 3.0 root hub
Bus 001 Device 006: ID 046d:c52b Logitech, Inc. Unifying Receiver
Bus 001 Device 004: ID 05e3:0610 Genesys Logic, Inc. Hub
Bus 001 Device 003: ID 413c:2107 Dell Computer Corp. KB212-B Quiet Key Keyboard
Bus 001 Device 005: ID 8087:0026 Intel Corp. AX201 Bluetooth
    ^^^        ^^^                           ^^^^^
Bus 001 Device 002: ID 103c:84fd HP TracerLED
Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
Bus 004 Device 001: ID 1d6b:0003 Linux Foundation 3.0 root hub
Bus 003 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
```

You can google the product name and find out e.g. that the AX201 is a Wi-Fi/Bluetooth combo device that supports Bluetooth 5.2.

Using the bus and device number, you can get more information:

```
$ sudo lsusb -v -s 001:005

Bus 001 Device 005: ID 8087:0026 Intel Corp. AX201 Bluetooth
Device Descriptor:
  bLength                18
  bDescriptorType         1
  bcdUSB               2.01
  bDeviceClass          224 Wireless
  bDeviceSubClass         1 Radio Frequency
  bDeviceProtocol         1 Bluetooth
  bMaxPacketSize0        64
  idVendor           0x8087 Intel Corp.
  idProduct          0x0026 AX201 Bluetooth
  bcdDevice            0.02
  iManufacturer           0 
  iProduct                0 
  iSerial                 0 
  bNumConfigurations      1
  Configuration Descriptor:
    bLength                 9
    bDescriptorType         2
    wTotalLength       0x00c8
    bNumInterfaces          2
    bConfigurationValue     1
    iConfiguration          0 
    bmAttributes         0xe0
      Self Powered
      Remote Wakeup
    MaxPower              100mA
    Interface Descriptor:
      bLength                 9
      ...
        bInterval               1
Binary Object Store Descriptor:
  bLength                 5
  bDescriptorType        15
  wTotalLength       0x000c
  bNumDeviceCaps          1
  USB 2.0 Extension Device Capability:
    bLength                 7
    bDescriptorType        16
    bDevCapabilityType      2
    bmAttributes   0x0000040e
      BESL Link Power Management (LPM) Supported
    BESL value     1024 us 
Device Status:     0x0001
  Self Powered
```

Without `sudo` it won't be able to open the device and won't be able to display everything it can about the device.

---

If you have a PCI, rather than USB, device, you'll have to use `lspci` rather than `lsusb`.

---

Irrespective of USB or PCI, you can use `lshw`:

```
$ sudo lshw -class communication
  *-usb:3                   
       description: Bluetooth wireless interface
       product: AX201 Bluetooth
       vendor: Intel Corp.
       physical id: e
       bus info: usb@1:e
       version: 0.02
       capabilities: bluetooth usb-2.01
       configuration: driver=btusb maxpower=100mA speed=12Mbit/s
...
```

Interestingly, Wi-Fi is in the `network` class while Bluetooth is in the `communication` class.

---

You can also use `bluetoothctl` to show information about the device:

```
$ bluetoothctl show
Controller B0:7D:64:56:0B:4D (public)
	Name: example-host
	Alias: example-host
	Class: 0x006c0104
	Powered: yes
	Discoverable: yes
	DiscoverableTimeout: 0x00000000
	Pairable: no
	UUID: A/V Remote Control        (0000110e-0000-1000-8000-00805f9b34fb)
	UUID: Handsfree Audio Gateway   (0000111f-0000-1000-8000-00805f9b34fb)
	UUID: PnP Information           (00001200-0000-1000-8000-00805f9b34fb)
	UUID: Audio Sink                (0000110b-0000-1000-8000-00805f9b34fb)
	UUID: Headset                   (00001108-0000-1000-8000-00805f9b34fb)
	UUID: A/V Remote Control Target (0000110c-0000-1000-8000-00805f9b34fb)
	UUID: Generic Access Profile    (00001800-0000-1000-8000-00805f9b34fb)
	UUID: Audio Source              (0000110a-0000-1000-8000-00805f9b34fb)
	UUID: Generic Attribute Profile (00001801-0000-1000-8000-00805f9b34fb)
	UUID: Device Information        (0000180a-0000-1000-8000-00805f9b34fb)
	Modalias: usb:v1D6Bp0246d0540
	Discovering: no
	Roles: central
	Roles: peripheral
Advertising Features:
	ActiveInstances: 0x00 (0)
	SupportedInstances: 0x06 (6)
	SupportedIncludes: tx-power
	SupportedIncludes: appearance
	SupportedIncludes: local-name
	SupportedSecondaryChannels: 1M
	SupportedSecondaryChannels: 2M
	SupportedSecondaryChannels: Coded
```

`bluetoothctl` is also the way to interact with the device from the command line:

```
Usage:
	bluetoothctl [--options] [commands]
Options:
	--agent 	Register agent handler: <capability>
	--monitor 	Enable monitor output
	--timeout 	Timeout in seconds for non-interactive mode
	--version 	Display version
	--help 		Display help
Commands:
	list		List available controllers
	show		Controller information
	select		Select default controller
	devices		List available devices
	paired-devices	List paired devices
	system-alias	Set controller alias
	reset-alias	Reset controller alias
	power		Set controller power
	pairable	Set controller pairable mode
	discoverable	Set controller discoverable mode
	discoverable-timeout	Set discoverable timeout
	agent		Enable/disable agent with given capability
	default-agent	Set agent as the default one
	advertise	Enable/disable advertising with given type
	set-alias	Set device alias
	scan		Scan for devices
	info		Device information
	pair		Pair with device
	cancel-pairing	Cancel pairing with device
	trust		Trust device
	untrust		Untrust device
	block		Block device
	unblock		Unblock device
	remove		Remove device
	connect		Connect device
	disconnect	Disconnect device
...
```

---

As we saw above, the Bluetooth interface is called `hci0`. And all the classic tools have names starting with `hci`. `hciconfig` is yet another option for displaying Bluetooth related information:

```
$ hciconfig -a
hci0:	Type: Primary  Bus: USB
	BD Address: B0:7D:64:56:0B:4D  ACL MTU: 1021:4  SCO MTU: 96:6
	UP RUNNING PSCAN ISCAN 
	RX bytes:1771 acl:0 sco:0 events:89 errors:0
	TX bytes:3805 acl:0 sco:0 commands:89 errors:0
	Features: 0xbf 0xfe 0x0f 0xfe 0xdb 0xff 0x7b 0x87
	Packet type: DM1 DM3 DM5 DH1 DH3 DH5 HV1 HV2 HV3 
	Link policy: RSWITCH SNIFF 
	Link mode: PERIPHERAL ACCEPT 
	Name: 'ghawkins-OMEN-25L-Desktop-GT12-0xxx'
	Class: 0x6c0104
	Service Classes: Rendering, Capturing, Audio, Telephony
	Device Class: Computer, Desktop workstation
	HCI Version: 5.1 (0xa)  Revision: 0x100
	LMP Version: 5.1 (0xa)  Subversion: 0x100
	Manufacturer: Intel Corp. (2)
```

Note: HCI stands for Host Controller Interface which "is a thin layer which transports commands and events between the host and controller elements of the Bluetooth protocol stack".

`hciconfig` etc. were deprecated in 2017 (see [here](https://wiki.archlinux.org/title/bluetooth#Deprecated_BlueZ_tools)) and these days, one should use `bluetoothctl` and `btmgmt`. We saw `bluetoothctl` above, `btmgmt` can also display information:

```
$ btmgmt info
Index list with 1 item
hci0:	Primary controller
	addr B0:7D:64:56:0B:4D version 10 manufacturer 2 class 0x6c0104
                           ^^^^^^^^^^
	supported settings: powered connectable fast-connectable discoverable bondable link-security ssp br/edr hs le advertising secure-conn debug-keys privacy configuration static-addr phy-configuration wide-band-speech 
	current settings: powered connectable discoverable ssp br/edr le secure-conn 
	name ghawkins-OMEN-25L-Desktop-GT12-0xxx
	short name 
hci0:	Configuration options
	supported options: public-address 
	missing options:
```

The version shown here allows you to map to more commonly used Core Specification versions using the [`core_version.yaml`](https://bitbucket.org/bluetooth-SIG/public/src/main/assigned_numbers/core/core_version.yaml) documenent published by the Bluetooth SIG. 10 is 0x0A in hex and this maps to 5.1:

```
...
- value: 0x09
   name: Bluetooth® Core Specification 5.0
 - value: 0x0A
   name: Bluetooth® Core Specification 5.1
 - value: 0x0B
   name: Bluetooth® Core Specification 5.2
```

Oddly, this contradicts the AX201 product page which says its a 5.2 device.

If you do `btmgmt --help`, you'll see it looks weirdly similar to `bluetoothctl` - both are part of the standard BlueZ Linux stack (`bluetoothctl` isn't a Debian specific addition as suggested [here](https://askubuntu.com/a/1311088/734204)). Why thet're so similar I don't know.

For more about them and the other BlueZ tools, see the ArchLinux Bluetooth [wiki page](https://wiki.archlinux.org/title/bluetooth).

---

The tool `inxi` can do the version mapping for you. It's not installed by default, so:

```
$ sudo apt install inxi
```

And then:

```
$ inxi -E -x
Bluetooth:
  Device-1: Intel AX201 Bluetooth type: USB driver: btusb v: 0.8
    bus-ID: 1-14:5
  Report: hciconfig ID: hci0 rfk-id: 0 state: up address: B0:7D:64:56:0B:4D
    bt-v: 3.0 lmp-v: 5.1
              ^^^^^^^^^^
```
