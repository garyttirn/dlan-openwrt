## Description

Firmware and basic setup scripts for the PLC interface.

## Usage
You have to build a firmware image containing the dlan-fw packages as it is not possible to distribute 
Firmware NVM/PIB files.

Device | Required PLC firmware package
---|---
dLAN pro 1200+ WiFi ac|dlan-fw-pro-1200plus-ac

After installing the new firmware packages use the init script ```/etc/init.d/plc``` to
start or stop the PLC interface. By default its enabled to be started automatically
at boot time. To disable it permanently call ```/etc/init.d/plc disable```,
reenable with ```/etc/init.d/plc enable```.

## PLC push button pairing (NOT TESTED)
Push button pairing is started by pressing the Home/dLAN button for max 3 seconds.
Pressing it again before the pairing is finished stops the pairing process.
Pressing the button for more than 10 seconds randomizes the NMK to leave the AVLN.
Pressing between 3s and 10s has no effect.

## PLC manual pairing using open-plc-utils

PLC settings are stored to /lib/firmware/plc/user.pib on OpenWRT.

This is what the settings look like after flashing OpenWRT :
```
root@OpenWrt:~# chkpib -v /lib/firmware/plc/user.pib
------- /lib/firmware/plc/user.pib -------
	PIB 0-0 21384 bytes
	MAC B8:BE:F4:7F:9C:29
	DAK 5C:A6:7E:27:DB:68:AE:67:B3:C6:6F:F5:A5:0F:F0:4F
	NMK 50:D3:E4:93:3F:85:5B:70:40:78:4D:F8:15:AA:8D:B7 (HomePlugAV)
	NID B0:F2:E6:95:66:6B:03
	Security level 0
	NET 
	MFG devolo dLAN 1200+ WiFi ac [MT2910;X]
	USR OpenWrt
	CCo Auto
	MDU N/A
```

open-plc-utils on a Linux PC are needed to fetch your PLC network config from an active PLC device.
https://github.com/qca/open-plc-utils

Scan the current PLC network, you need to be connected to the same Ethernet segment as the PLC device is via Ethernet/WLAN (not via PLC).
Interface eth0 depends on your PC.
```
sudo ./plctool -i eth0 -m all
eth0 FF:FF:FF:FF:FF:FF Fetch Network Information
eth0 30:D3:2D:F3:89:6F Found 1 Network(s)

source address = 30:D3:2D:F3:89:6F

	network->NID = 6E:ED:DF:82:15:A3:03
	network->SNID = 2
	network->TEI = 1
	network->ROLE = 0x02 (CCO)
	network->CCO_DA = 30:D3:2D:F3:89:6F
	network->CCO_TEI = 1
	network->STATIONS = 0
```

Fetch the config to a tmp-file
```
sudo ./plctool -i eth0 -p /tmp/remote.pib 30:D3:2D:F3:89:6F
eth0 30:D3:2D:F3:89:6F Read Module from Memory

chkpib -v /tmp/remote.pib
------- /tmp/remote.pib -------
	PIB 0-0 21384 bytes
	MAC 30:D3:2D:F3:89:6F
	DAK 26:CA:8C:A2:D9:08:5F:DD:B5:A6:62:22:2A:17:56:41
	NMK DE:CC:85:24:A5:22:17:71:5F:56:54:3C:9A:6B:3A:F8
	NID 6E:ED:DF:82:15:A3:03
	Security level 0
	NET 
	MFG devolo dLAN 1200+ [MT2853]
	USR Single
	CCo Auto
	MDU N/A
```

Update the Network Key on OpenWRT PLC
```
root@OpenWrt:~# modpib -N DE:CC:85:24:A5:22:17:71:5F:56:54:3C:9A:6B:3A:F8 /lib/firmware/plc/user.pib
root@OpenWrt:~# chkpib -v /lib/firmware/plc/user.pib
------- /lib/firmware/plc/user.pib -------
	PIB 0-0 21384 bytes
	MAC B8:BE:F4:7F:9C:29
	DAK 5C:A6:7E:27:DB:68:AE:67:B3:C6:6F:F5:A5:0F:F0:4F
	NMK DE:CC:85:24:A5:22:17:71:5F:56:54:3C:9A:6B:3A:F8
	NID 6E:ED:DF:82:15:A3:03
	Security level 0
	NET 
	MFG devolo dLAN 1200+ WiFi ac [MT2910;X]
	USR OpenWrt
	CCo Auto
	MDU N/A
```

Restart PLC interface
```
root@OpenWrt:~# /etc/init.d/plc restart
```

Check PLC network (openwrt 22.03 and fw4 will only work if plctool is used on br-lan interface instead of eth) :

```
root@OpenWrt:~# plctool -i eth0.1 -m all 
eth0.1 FF:FF:FF:FF:FF:FF Fetch Network Information
eth0.1 B8:BE:F4:7F:9B:29 Found 1 Network(s)

source address = B8:BE:F4:7F:9C:29

	network->NID = 6E:ED:DF:82:15:A3:03
	network->SNID = 2
	network->TEI = 8
	network->ROLE = 0x00 (STA)
	network->CCO_DA = 30:D3:2D:E3:89:6F
	network->CCO_TEI = 1
	network->STATIONS = 1

		station->MAC = 30:D3:2D:E3:89:6F
		station->TEI = 1
		station->BDA = 10:C3:7B:9C:A7:C6
		station->AvgPHYDR_TX = 146 mbps Alternate
		station->AvgPHYDR_RX = 158 mbps Alternate

eth0.1 30:D3:2D:F3:89:6F Found 1 Network(s)

source address = 30:D3:2D:F3:89:6F

	network->NID = 6E:ED:DF:82:15:A3:03
	network->SNID = 2
	network->TEI = 1
	network->ROLE = 0x02 (CCO)
	network->CCO_DA = 30:D3:2D:F3:89:6F
	network->CCO_TEI = 1
	network->STATIONS = 1

		station->MAC = B8:BE:F4:7F:9C:29
		station->TEI = 8
		station->BDA = B8:BE:F4:7F:9C:28
		station->AvgPHYDR_TX = 162 mbps Alternate
		station->AvgPHYDR_RX = 146 mbps Alternate
```

## License

Firmware NVM/PIB files in qca/:
Copyright (c) Qualcomm Atheros, Inc.
See LICENSE in qca/*

All other files:
Copyright (c) 2017 devolo AG
