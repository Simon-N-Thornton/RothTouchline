# RothTouchline Heating controller


### Background
Roth make a series of controllers for underfloor heating, the older versions only support a wired ethernet control port, newer versions support wireless. These are coupled with wireless (not WiFI) thermostats for control, each controller can have multiple thermostats (one per room).  The older versions have a web interface based on a java jnlp, which unfortunately does not work on modern browsers as the Java web launch feature has been removed. For those with Android or IoS devices a Touchline application exists to allow you to set the thermostats remotely. My challenge was to integrate this heating controller into my home assistant environment.

![alt text](https://www.roth-uk.com/fileadmin/user_upload/Roth_North_Europe/Images_for_Roth_North_Europe/Danmark/Produkter_images/Touchline/Touchline_kontrol_enhed_uden_LAN.jpg "Roth Touchline PL")&nbsp;![alt text](https://www.roth-uk.com/fileadmin/user_upload/Roth_North_Europe/Images_for_Roth_North_Europe/Danmark/Produkter_images/Touchline/Touchline_stregtegning_382x79.jpg "LAN Module")

### Network
To connect the wired-only Roth controller I used a Vonets VAR11N_300 Wireless/Ethernet bridge, which is a matchbox sized device.


### API Interface
Data from the controller can be accessed using the web interface endpoints readVal.cgi and writeVal.cgi. e.g.  

Read  Value: http://xxx.xxx.xxx.xxx/readVal.cgi?variable  
Write Value: http://xxx.xxx.xxx.xxx/writeVal.cgi?variable=value  
  
IP address: xxx.xxx.xxx.xxx  
  
### Device Parameters  

CD = Global Variables  
Rx = Regulator  
Gx = Thermostat  
HW = Network Interface  
  
| Variable                 | Values            | Description |  
| ---                      | ---               | --- | 
| CD.uname                 |                   | Username  (only certain FW versions) |  
| CD.upass                 | 1234              | User interface password (default 1234) |  
| STELL-APP                | 1.42              | Internal Version number (RO) |  
| STELL-BL                 | 1.20              | Module version |  
| STM-BL                   | 1.20              | MOdule version |  
| hw.IP                    | xxx.xxx.xxx.xxx   | IP Address of Device |  
| hw.NM                    | 255.255.255.0     | Netmask of IP |  
| hw.Addr                  | 5C-AB-23-DF-C9-FA | MAC address of Interface |  
| hw.DNS1                  | dd1.dd1.dd1.dd1   | IP address of DNS entry #1 |  
| hw.DNS2                  | dd2.dd3.dd2.dd2   | IP address of DNS entry #2 |  
| hw.GW                    | zzz.zzz.zzz.zzz   | Default route |  
| hw.HostName              | ROTH-DFC9FA       | Hostname (default ROTH-last_6_digits_of_MAC) |  
| totalNumberOfDevices     | 4                 | Number of thermostats attached, 4 indicates thermostats 0-3 |  
| numberOfSlaveControllers | 0                 | Number of slave controllers attach |  
| VPI.href                 | http://myroth.ininet.ch/remote/t_uniqueID/ | URL of remote access point, see uniqueID below |  
| VPI.state                | 99                | Remote access point status |  
| isMaster                 | 1                 | Is this a master or slave (push master button for this to work) |  
| Status                   | Server: Roth/1.0 (powered by SpiderControl TM), CGI=0, ILR=0, V.1.0, ILR2=0, V.2.00, ILR3=1, V.1.00 | Status of webserver components |  
|                          |                   |  
| R0.DateTime              | EPOCH datetime    | Date/time since 1970 e.g. 1703956882 or " Sat Dec 30 15:33:54 CET 2023", can be updated |  
| R0.ErrorCode             | 0                 | Current error |  
| R0.OPModeRegler          | 0                 | ??? |  
| R0.Safety                | 0                 | ??? |  
| R0.SystemStatus          | 0                 | System status, 0=off, 1=running |  
| R0.Taupunkt              | 0                 | ??? |  
| R0.WeekProgWarn          | 1                 | ??? |  
| R0.kurzID                | 69                | same as Gx.kurzID
| R0.numberOfPairedDevices | 4                 | Number of paired devices, same as 'totalNumberOfDevices' |  
| R0.uniqueID              | 63BF710649F95434  | Unique identifier, used to construct VPI.href | 


### Thermostat parameters  
These parameters can be updated using the writeVal.cgi?variable=value  
e.g.  
      http://xxx.xxx.xxx.xxx/writeVal.cgi?variable=value  

Gx indicates the thermostat index (0 to totalNumberofDevices-1)  

| Variable               | Values            | Description |  
| ---                    | ---               | --- | 
| Gx.name                | <Whatever>        | Name of thermostat, if set |  
| Gx.RaumTemp            | 2094              | Room temperature, 20.94 |  
| Gx.SollTemp            | 2000              | Room set temperature, 20.00 |  
| Gx.SollTempMaxVal      | 2600              | Max temperature allowed for SollTemp, 26.00 |  
| Gx.SollTempMinVal      | 1600              | Min temperature allowed for SollTemp, 16.00 |  
| Gx.TempSIUnit          | 0                 | Temperature scale, 0=C, 1=F |  
| Gx.WeekProgEna         | 1                 | Weekly mode enabled |  
| Gx.OPMode              | 0-2               | Operation Mode: 0=Normal, 1=Night, 2=Vacation |  
| Gx.OPModeEna           | 0-1               | Thermostat enabled (1) or disabled (0) |  
| Gx.kurzID              | 1                 | ??? Same for all thermostats |  
| Gx.ownerKurzID         | 69                | ??? Same for all thermostats |  

### Examples

| Purpose | cmdline |  
| ---     | --- |  
| Read current date/time | curl http://xxx.xxx.xxx.xxx/readVal.cgi?R0.DateTime |  
| Set Date/Time | curl http://xxx.xxx.xxx.xxx/writeVal.cgi?R0.DateTime=$(date +%s) |  
| Set name of thermostat 0 | curl http://xxx.xxx.xxx.xxx/writeVal.cgi?G0.name=My_Thermostat |  
| Set Temp of thermostat 0 to 20.54 C | curl http://xxx.xxx.xxx.xxx/writeVal.cgi?G0.SollTemp=2054 |  
| Read Temp of thermostat 0 | curl http://xxx.xxx.xxx.xxx/readVal.cgi?G0.SollTemp |  
| Set thermostat 0 mode to night | curl http://xxx.xxx.xxx.xxx/writeVal.cgi?G0.OPMode=1 |  

### Locating API endpoints
The easiest way to find the API endpoints for the older controllers, not the SL version, is to download the firmware, unpack it and examine the file "Roth.tcr".  

![alt text](https://www.roth-uk.com/fileadmin/user_upload/Roth_North_Europe/Images_for_Roth_North_Europe/UK/Images/Support/Firmware/Kompatibilitetsskema_Touchline_firmware_alle_sprog_20191007_UK_v2.jpg "Touchline controllers")

Source: https://www.roth-uk.com/support/software-and-firmware-updates  

The "Roth.tcr" file contains a list of the (potential) API calls that can be used with readVal.cgi and writeVal.cgi calls. Not all of these are implemented in the Touchline build and the unimplemented ones return a 404 error when called.  

```
CD.reset;CD.reset; ; ; ; ; ; ; ; ; ;
CD.save;CD.save; ; ; ; ; ; ; ; ; ;
CD.submit;CD.submit; ; ; ; ; ; ; ; ; ;
CD.submit.err;CD.submit.err; ; ; ; ; ; ; ; ; ;
CD.uname;CD.uname; ; ; ; ; ; ; ; ; ;
CD.upass;CD.upass; ; ; ; ; ; ; ; ; ;
CD.ureg;CD.ureg; ; ; ; ; ; ; ; ; ;
G#CO_myGx#.TempSIUnit;G#CO_myGx#.TempSIUnit; ; ; ; ; ; ; ; ; ;
G#CO_myGx#.WeekProg;G#CO_myGx#.WeekProg; ; ; ; ; ; ; ; ; ;
G0+#COFF_devicePageOffset#.TempSIUnit;G0+#COFF_devicePageOffset#.TempSIUnit; ; ; ; ; ; ; ; ; ;
G0+@COFF_configPageOffset@.kurzID;G0+@COFF_configPageOffset@.kurzID; ; ; ; ; ; ; ; ; ;
G0+@COFF_configPageOffset@.name;G0+@COFF_configPageOffset@.name; ; ; ; ; ; ; ; ; ;
.
.
R0.DateTime;R0.DateTime; ; ;Int; ; ; ; ; ; ;
R0.ErrorCode;R0.ErrorCode; ; ; ; ; ; ; ; ; ;
R0.OPModeRegler;R0.OPModeRegler; ; ; ; ; ; ; ; ; ;
R0.Safety;R0.Safety; ; ; ; ; ; ; ; ; ;
R0.SystemStatus;R0.SystemStatus; ; ; ; ; ; ; ; ; ;
R0.Taupunkt;R0.Taupunkt; ; ; ; ; ; ; ; ; ;
R0.WeekProgWarn;R0.WeekProgWarn; ; ; ; ; ; ; ; ; ;
R0.kurzID;R0.kurzID; ; ; ; ; ; ; ; ; ;
R0.numberOfPairedDevices;R0.numberOfPairedDevices; ; ; ; ; ; ; ; ; ;
.
.
```
Note: the '+#COFF_devicePageOffset#' translates into the index of the thermostat  


### API Script
The sum of all this has been captured in a bash script called 'rothread.sh' that 
- Allows entry of temperatures in dsecimal x.y format i.e. 18.54 instead of 1854
- Implements a status print of all variables
- Logs the output to files in /var/tmp
- Builds an arrays of field names/values to allow this to be called from other scripts

e.g.

| cmd line | Description |  
| ---      | ---         |  
| rothread.sh -h | Show help |  
| rothread.sh -s | Show status of all variables |  
| rothread.sh -w G0.OPMode=1 | Set Thermostat #0 to Night mode |  
| rothread.sh -w G1.SollTemp=19.54 | Set required Temperature to 19.54 on Thermostat #2 |  
| rothread.sh -w R0.Datetime=$(date +%s) | Set controller Date/Time to current on Linux |  

### Roth Touchline SL API
The firmware file format for the newer Touchline SL controllers has a different format to the older Touchline BL/PL controllers. The file appears to be encoded and does not reveal anything about the internal contents.  

I have no way to test this script against the newer Touchline SL; if you have one of these controllers please let me know your results.

![alt text](https://www.roth-uk.com/fileadmin/user_upload/Roth_North_Europe/Images_for_Roth_North_Europe/UK/Images/Support/Firmware/Kompatibilitetsskema_Touchline_SL_firmware_UK_20241202.png "Roth Touchline SL comparison")

