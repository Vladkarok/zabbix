# Discovery iLO sersors with perl scripts

## Description
This template with two perl scripts can perform discovery of sensors like temperature and fan speed in IPMI iLO

## Overview
There is already build-in IPMI discovery https://git.zabbix.com/projects/ZBX/repos/zabbix/browse/templates/server/chassis_ipmi but it has issue with **discrete** sensors, for example executing *ipmitool* command can get the following result:

```bash
ipmitool -U USERNAME -P PASSWORD -H IPMI_HOST -I lanplus sensor get "Fan 1"
```
output:

```
Locating sensor record...
Sensor ID              : Fan 1 (0x1a)
 Entity ID             : 7.1
 Sensor Type (Discrete): Fan
 Sensor Reading        : 5.880 percent
 States Asserted       : Fan
                         [Transition to Running]
```
But zabbix (at this moment) will not get value `5.880` because sensor type is **discrete**  
These scripts can extract such values.  
Tested in Zabbix Server v.**6.x** \
HP iLO4 firmware **2.81** \
ipmi-sensors - **1.6.9**
## Installation
### iLO side
Create simple user (without administration priviledges) in iLO

### Zabbix server side
#### SSH to Zabbix host:
Edit configuration file `/etc/zabbix/zabbix_server.conf` for `StartIPMIPollers` value of `1` or more

```conf
StartIPMIPollers=1
```
and specify external script directory (`/usr/lib/zabbix/externalscripts` default)

```conf
ExternalScripts=/usr/lib/zabbix/externalscripts
```

Place `ilo_discovery.pl` and `ipmi_proliant.pl` scripts (source located in **`perl_scripts`** directory in this repository) in **ExternalScripts** directory, give them execution rights

```
chmod +x /usr/lib/zabbix/externalscripts/i*
```
Install **freeipmi-tools** https://www.gnu.org/software/freeipmi/

For apt-systems run:
```
apt install freeipmi-tools
```
Check if zabbix host can connect to iLO and get values (where **ILO_ADRESS**, **PASSWORD**, **USERNAME** are your values)
via FreeIPMI:
```bash
ipmi-sensors -D LAN2_0 -h ILO_ADDRESS -u USERNAME  -l USER -p PASSWORD -W discretereading --no-header-output --quiet-cache --sdr-cache-recreate --comma-separated-output --entity-sensor-names
```
sample output:
```m
0,System Chassis 1 UID Light,OEM Reserved,N/A,N/A,'OEM Event = 0000h'
1,System Chassis 2 Health LED,OEM Reserved,N/A,N/A,'OEM Event = 0000h'
2,Processor Module VRM 1,Power Unit,N/A,N/A,'Device Inserted/Device Present'
3,Power Supply Power Supply 1,Power Supply,N/A,N/A,'Presence detected'
```

and via script

```bash
/usr/lib/zabbix/externalscripts/ilo_discovery.pl USER PASSWORD ILO_ADRESS sensor temp numeric
```
sample output:

```json
{
        "data":[
                {
                        "{#CLASS}":"sensor",
                        "{#KEY}":"Air Inlet 01-Inlet Ambient",
                        "{#SECTION}":"Temperature",
                        "{#TYPE}":"numeric",
                        "{#MEASURE}":"C"},
                {
                        "{#CLASS}":"sensor",
                        "{#KEY}":"Processor 02-CPU",
                        "{#SECTION}":"Temperature",
                        "{#TYPE}":"numeric",
                        "{#MEASURE}":"C"},
```
#### Zabbix frontend:

Import template `template_ipmi_script_discovery.yaml`
Create host, attach template, fill the macroses


## Macros used

|Name|Description|Default|
|----|-----------|-------|
|{$ILO}|Address of iLO|<p>-</p>|
|{$IPMI.USER}|IPMI user|<p>-</p>|
|{$IPMI.PASSWORD}|IPMI password|<p>-</p>|

## Discovery rules

|Name|Description|Type|Key and additional info|
|----|-----------|----|----|
|Discovery iLO Chassis fault flags |<p>-</p> |External check |ilo_discovery.pl[{$IPMI.USER},{$IPMI.PASSWORD},{$ILO},chassis,fault,discrete] |
|Discovery iLO Chassis Power State |<p>-</p> |External check |ilo_discovery.pl[{$IPMI.USER},{$IPMI.PASSWORD},{$ILO},chassis,"system power"] |
|Discovery iLO Drives|<p>-</p> |External check |ilo_discovery.pl[{$IPMI.USER},{$IPMI.PASSWORD},{$ILO},sensor,"disk",discrete] |
|Discovery iLO Fans |<p>-</p> |External check |ilo_discovery.pl[{$IPMI.USER},{$IPMI.PASSWORD},{$ILO},sensor,"fan",numeric] |
|Discovery iLO Memory State |<p>-</p> |External check |ilo_discovery.pl[{$IPMI.USER},{$IPMI.PASSWORD},{$ILO},sensor,"memory",discrete] |
|Discovery iLO Power Meter |<p>-</p> |External check |ilo_discovery.pl[{$IPMI.USER},{$IPMI.PASSWORD},{$ILO},sensor,"power meter",numeric] |
|Discovery iLO Power Supplies Load |<p>-</p> |External check |ilo_discovery.pl[{$IPMI.USER},{$IPMI.PASSWORD},{$ILO},sensor,"power supply",numeric] |
|Discovery iLO Power Supplies State |<p>-</p> |External check |ilo_discovery.pl[{$IPMI.USER},{$IPMI.PASSWORD},{$ILO},sensor,"power suppl",discrete] |
|Discovery iLO Temperatures |<p>-</p> |External check |ilo_discovery.pl[{$IPMI.USER},{$IPMI.PASSWORD},{$ILO},sensor,temp,numeric] |
|Discovery iLO VRM State |<p>-</p> |External check |ilo_discovery.pl[{$IPMI.USER},{$IPMI.PASSWORD},{$ILO},sensor,VRM,discrete] |

## Items collected

|Group|Name|Description|Type|Key and additional info|
|-----|----|-----------|----|---------------------|
|System |Board Serial Number |<p> - </p> |External check |`ipmi_proliant.pl[{$IPMI.USER},{$IPMI.PASSWORD},'FRU Board Serial Number',fru,{$ILO}]`|
|System |Chassis Serial Number |<p> - </p> |External check |`ipmi_proliant.pl[{$IPMI.USER},{$IPMI.PASSWORD},'FRU Chassis Serial Number',fru,{$ILO}]`|
|System |Chassis Type |<p> - </p> |External check |`ipmi_proliant.pl[{$IPMI.USER},{$IPMI.PASSWORD},'FRU Chassis Type',fru,{$ILO}]`|
|System |Server Model Name |<p> - </p> |External check |`ipmi_proliant.pl[{$IPMI.USER},{$IPMI.PASSWORD},'FRU Board Product Name',fru,{$ILO}]`|
|System |iLO Firmware Version |<p> - </p> |External check |`ipmi_proliant.pl[{$IPMI.USER},{$IPMI.PASSWORD},'Firmware Revision',bmc,{$ILO}]`|
|Availability |iLO ping |<p> - </p> |Simple check |`icmpping[{$ILO},10,,,]`|
|Health |{#KEY} flag |<p>Discovery iLO Chassis fault flags </p> |External check |`ipmi_proliant.pl[{$IPMI.USER},{$IPMI.PASSWORD},'{#KEY}',chassis,'{$ILO}']`|
|Health |{#KEY} State |<p>Discovery iLO Chassis Power State </p> |External check |`ipmi_proliant.pl[{$IPMI.USER},{$IPMI.PASSWORD},'{#KEY}',chassis,{$ILO}]`|
|Health |{#KEY} Status |<p>Discovery iLO Drives </p> |External check |`ipmi_proliant.pl[{$IPMI.USER},{$IPMI.PASSWORD},"{#KEY}",sensor,'{$ILO}',discrete]`|
|Health |{#KEY} State |<p>Discovery iLO Fans </p> |External check |`ipmi_proliant.pl[{$IPMI.USER},{$IPMI.PASSWORD},'{#KEY}',sensor,'{$ILO}',discrete]`|
|Health |{#KEY} Value |<p>Discovery iLO Fans </p> |External check |`ipmi_proliant.pl[{$IPMI.USER},{$IPMI.PASSWORD},'{#KEY}',sensor,'{$ILO}',{#TYPE}]`|
|Health |{#KEY} State |<p>Discovery iLO Memory State </p> |External check |`ipmi_proliant.pl[{$IPMI.USER},{$IPMI.PASSWORD},'{#KEY}',sensor,"{$ILO}",discrete]`|
|Health |{#KEY} State |<p>Discovery iLO Memory State </p> |External check |`ipmi_proliant.pl[{$IPMI.USER},{$IPMI.PASSWORD},'{#KEY}',sensor,"{$ILO}",discrete]`|