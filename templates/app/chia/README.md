# Chia log monitoring by Zabbix Agent in Windows

## Overview  

Tested on Zabbix version: **6.2**, Chia version **1.6.1**.  
This template extracts data from the Chia log file using the Zabbix Agent (active mode). Also, it has built-in graphs on each item (but I recommend using dashboards for visualization) and trigger when the *time to find the proof* is long.

## Setup

Log file monitoring requires Zabbix Agent in **active** mode, so it needs to be installed with a configuration that will allow active checks:
- the same **hostname** in Zabbix Agent configuration and related Host in Zabbix frontend on your server;
- the **IP address/domain name** of your *Zabbix server active* directive in *Zabbix Agent configuration*  
You should be able to connect **from** monitored host **to** Zabbix server and open the 10051 port on the Zabbix server to receive connections from Zabbix Agent.  

Packages for Zabbix Agents can be found here https://www.zabbix.com/download_agents 
<br><br>
### In Windows

Enable *INFO* **log_level** in your Chia config file (C:\Users\YOUR_USERNAME\\.chia\mainnet\config\config.yaml

```yml
    log_level: WARNING
```
to
```yml
    log_level: INFO
```

and restart the Chia processes.

### In Zabbix server

Import the template from this repository to your Zabbix Server (Configuration - Templates - Import).
<br><br>

### Macros used

|Name|Description|Default|
|----|-----------|-------|
|{$SEARCH_REGEX} |<p>Regular expression to extract needed lines from log</p> |`(\d+) plots were eligible for farming .+\.\.\. Found (\d+) proofs\. Time: (\d+\.\d+) s\. Total (\d+) plots$` |
|{$USERNAME} |<p>Username of the user where Chia is installed</p> |`CHANGE_ME` |

**!!!**
>**Note! Put the username of your account instead of `CHANGE_ME` word.**  

**!!!**
## Items collected

|Name|Description|Key|
|----|-----------|---------------------|
|Plots eligible for farming |<p>Number of plots were eligible for farming.</p> |`log[C:\Users\{$USERNAME}\.chia\mainnet\log\debug.log,"{$SEARCH_REGEX}",,,skip,\1,,]` |
|Proofs found |<p>Number of found proofs.</p> |`log[C:\Users\{$USERNAME}\.chia\mainnet\log\debug.log,"{$SEARCH_REGEX}",,,skip,\2,,]` |
|Proof time |<p>Time which takes to find the proof.</p> |`log[C:\Users\{$USERNAME}\.chia\mainnet\log\debug.log,"{$SEARCH_REGEX}",,,skip,\3,,]` |
|Total plots |<p>Total number of plots.</p> |`log[C:\Users\{$USERNAME}\.chia\mainnet\log\debug.log,"{$SEARCH_REGEX}",,,skip,\4,,]` |

## Triggers

|Name|Description|Expression|Severity|Additional info|
|----|-----------|----|----|----|
|Proof searching time too long |<p>Proof searching time too long. Consider some investigation.</p> |`last(/Windows Chia log monitoring/log[C:\Users\{$USERNAME}\.chia\mainnet\log\debug.log,"{$SEARCH_REGEX}",,,skip,\3,,],#2)>12s` |AVERAGE |<p>Manual close: YES</p> |

## Used materials

https://www.zabbix.com/documentation/current/en/manual/config/items/itemtypes/zabbix_agent#log-data

https://regex101.com/

https://www.youtube.com/watch?v=3ljYwiVt1CA