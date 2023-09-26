# Linux CPU per core by Zabbix agent

## Overview  

Template with LLD of CPU cores with gathering stats for each core separately. Can be useful in multi-core setups when for example you looking at 15% total cpu utilization but some application is responding slowly but is appears one core is just bumping out 100% and rest are idling.<br>Most of the info was taken from [official template](https://git.zabbix.com/projects/ZBX/repos/zabbix/browse/templates/os/linux)

## Requirements

Zabbix version: 6.2 and higher.

## Configuration

> Zabbix should be configured according to the instructions in the [Templates out of the box](https://www.zabbix.com/documentation/current/en/manual/config/templates_out_of_the_box) section

## Setup

Zabbix agent required. No additional configuration.

### Macros used

|Name|Description|Default|
|----|-----------|-------|
|`{$CPU_CORE.UTIL.CRIT}`|Critical CPU Core utilization for trigger|`95`

### LLD rule **CPU Core Discovery**

|Name|Description|Type|Key and additional info|
|----|-----------|----|-----------------------|
|CPU Core Discovery|Discovers separate cores of CPU|Zabbix agent|system.cpu.discovery|

### Item prototypes for **CPU Core Discovery**

|Name|Description|Type|Key and additional info|
|----|-----------|----|-----------------------|
|CPU guest nice time by Core {#CPU.NUMBER}|Time spent running a niced guest (virtual CPU for guest operating systems under the control of the Linux kernel).|Zabbix agent|system.cpu.util[{#CPU.NUMBER},guest_nice]|
|CPU guest time by Core {#CPU.NUMBER}|Guest  time (time  spent  running  a  virtual  CPU  for  a  guest  operating  system).|Zabbix agent|system.cpu.util[{#CPU.NUMBER},guest]|
|CPU idle time by Core {#CPU.NUMBER}|The time the CPU has spent doing nothing.|Zabbix agent|system.cpu.util[{#CPU.NUMBER},idle]|
|CPU interrupt time by Core {#CPU.NUMBER}|The amount of time the CPU has been servicing hardware interrupts.|Zabbix agent|system.cpu.util[{#CPU.NUMBER},interrupt]|
|CPU iowait time by Core {#CPU.NUMBER}|Amount of time the CPU has been waiting for I/O to complete.|Zabbix agent|system.cpu.util[{#CPU.NUMBER},iowait]|
|CPU nice time by Core {#CPU.NUMBER}|The time the CPU has spent running users' processes that have been niced.|Zabbix agent|system.cpu.util[{#CPU.NUMBER},nice]|
|CPU softirq time by Core {#CPU.NUMBER}|The amount of time the CPU has been servicing software interrupts.|Zabbix agent|system.cpu.util[{#CPU.NUMBER},softirq]|
|CPU steal time by Core {#CPU.NUMBER}|The amount of CPU 'stolen' from this virtual machine by the hypervisor for other tasks (such as running another virtual machine).|Zabbix agent|system.cpu.util[{#CPU.NUMBER},steal]|
|CPU system time by Core {#CPU.NUMBER}|The time the CPU has spent running the kernel and its processes.|Zabbix agent|system.cpu.util[{#CPU.NUMBER},system]|
|CPU user time by Core {#CPU.NUMBER}|The time the CPU has spent running users' processes that are not niced.|Zabbix agent|system.cpu.util[{#CPU.NUMBER},user]|
|CPU utilization by Core {#CPU.NUMBER}|<p>CPU utilization by core in %.</p>|Dependent item|system.cpu.util[{#CPU.NUMBER}]<p>**Preprocessing**</p><ul><li><p>JavaScript: `//Calculate utilization<br>return (100 - value)`</p></li></ul>|

### Trigger prototypes for **CPU Core Discovery**

|Name|Description|Expression|Severity|Dependencies and additional info|
|----|-----------|----------|--------|--------------------------------|
|High utilization on Core {#CPU.NUMBER}|CPU Core utilization is too high.|min(/Linux CPU per core by Zabbix agent/system.cpu.util[{#CPU.NUMBER}],5m)>{$CPU_CORE.UTIL.CRIT}|Warning|**Manual close**: Yes|