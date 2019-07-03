# Serial Communication API for LakeWest Mini Streamer

-get command returns push

-set command returns push to confirm new value 

-push command to Mini Streamer is not a valid cmd, its only pushed back by Mini Streamer to return/confirm set value.

Terms definition:
Streamer Software - any software running on LakeWest Mini Streamer hardware using this open protocol
Mini Streamer - LakeWest hardware - connected onto CM3 UART port

UART is running at 115200kbps, no parity, 1 stop bit. Using pins GPIO32 (TX to MCU) and GPIO33 (RX from MCU).

WHITE network LED is connected on pin GPIO37, RED network LED is connected on pin GPIO38. 
Pin GPIO39 is MCU reset and must be set as input, unless UART bootloader cant be accesed and needs to be forced with pin GPIO35. GPIO34 shall be HIGH when CM3 finished booting and should go LOW when its safe to cut off CM3 power supply.

GPIO43 is LAN chip reset, most rpi firmwares take care of that (just needs to be defined in dtblob). 
GPIO40 when low shuts down the WiFi radio (by default must be forced high). 
GPIO41 is a wake pin from the WiFi module (its purpose is not really known RN).
            
#### USB DAC port

```shell
[getUSB]{}
```

```shell
[setUSB]{activeRoute:1,powerShutdown:0}
```

```shell
[pushUSB]{activeRoute:1,powerShutdown:0}
```
activeRoute:

	0   Streamer - DAC USB is routed to the CM3 USB host port,	
	1   USB pass-thru - cleaner mode, DAC USB is routed to back panel device USB,		
	2 - 255 reserved for future use.
  
powerShutdown:

	0   +5.2V USB output clean power for DAC is enabled,	
	1   disables power for USB dac, keeps data going (user has to power DAC from different supply)		
	2 - 255 reserved for future use.

#### ADC REEDBACK

```shell
[getADC]{port:1}
```

```shell
[pushADC]{port:1,voltage:522,current:496,power:259}
```

port:

	0   "USB1" (back DAC),	
	1   "USB2" (back universal),	
  	2   "USB3" (front universal),	
  	3   "PWR"
	
	voltage - return voltage with 2 decimal point precision - e.g. 522 is 5.22V, 1856 is 18.56V etc..
	current - recurns current in mA - e.g. 496 is 496mA
	power - returns power with 2 decimal places - 259 is 2.59W

#### POWER/UNIT STATUS

```shell
[getPowerStatus]{}
```

```shell
[pushPowerStatus]{status:0}
```

status:

	0   Everything is OK,	
	1   Input voltage too low, audio performance could be degraded, connect 18V power supply
  	2   Unit failure, input regulator could be damaged
  	
	
#### Mini Streamer HW BOOT CONTROL

```shell
[pushBoot]{boot:0}
```

```shell
[setBoot]{boot:0}
```

boot:

	0   note defined,	
	1   puts CM3 into USB bootloader mode where new Streamer Software image can be uploaded over USB	
	
Bootloader mode has to be switched off by user presing "USB" toggle button on the unit front panel

#### WATCHDOG

```shell
[getWatchdog]{}
```

```shell
[setWatchdog]{timeout:0}
```

```shell
[pushWatchdog]{timeout:0}
```

Watchdog, expects pulse on GPIO34 to reset its timer, about 1ms pulse width is ok (probably much less). By default disabled, shall be enabled when software gets to the stage where it can reliably generate pulses. Can be disabled for task like memory update etc. 

timeout:
	watchdig timeout in seconds, maximum 30s, if exceeded, 30s is set and timer activated
	
	0 - disables timer
	1 to 30 - time in seconds

#### POWER OFF CONTROL

Is pushed from the Mini Streamer HW to Streamer Software to signal that user has switched off the unit with hardware front panel toggle. Streamer Software then needs to start shutting down. After its ready to cut the power off, it sets GPIO34 low and power is going to be cut 3 seconds later.

If GPIO34 is not put low within 60 seconds, the unit will be shut anyway. If GPIO34 is low when shutdown is commanded (shall go high when Streamer Software has booted up), unit is shut down immediately. 

```shell
[getTurnOff]{}
```

```shell
[setTurnOff]{cmd:0}
```

```shell
[pushTurnOff]{cmd:0}
```
cmd:

	0 - not valid
	1 - unit is shutting down (either set from Streamer Software, or pushed from the Mini Streamer HW)
	
#### POWER ON STATE

```shell
[getPowerOnstate]{}
```

```shell
[setPowerOnstate]{state:0}
```

```shell
[pushPowerOnstate]{state:0}
```
state:

	0 - normal operation, unit will be OFF after powe loss
	1 - remembers the last power state (and will act accordingly after power loss)
	2 - unit will always switch on after power loss recovery (embedded units in walls and so on)

	
#### HDMI REQUEST

```shell
[getHDMIstate]{}
```

```shell
[setHDMIstatus]{enabled:0}
```

```shell
[pushHDMIstatus]{enabled:0}
```

Disables or enables HDMI output.

#### SOC TEMP

```shell
[getSoCtemp]{}
```

```shell
[setSoCtemp]{temp:65}
```

```shell
[pushSoCtemp]{temp:65}
```
SoC Temperature, host should reply by set cmd on get cmd from MCU.

#### Mini Streamer HW VERSION

```shell
[getBoardVersion]{}
```

```shell
[pushBoardVersion]{boardRev:12,firmwareRev:11,boardType:0}
```
boardType:
	
	0 - lakewest mini
	1 - project S2
	2 - Stack audio
	
#### STREAMER VERSION

```shell
[getStreamerVersion]{}
```

```shell
[pushStreamerVersion]{version:1223}
```

```shell
[setStreamerVersion]{version:1223}
```
#### TODO:

Define UART bootloader for onboard firmware upgrade
