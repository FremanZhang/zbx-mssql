# zbx-mssql

Zabbix Template for monitoring and collecting Microsoft SQL Server statistics.
Most requests are done via Windows performance counters (`perf_counter`), some via PowerShell and, optionaly, via ODBC.


![CPU/Memory Load](https://github.com/sfuerte/zbx-mssql/blob/master/images/zbx_mssql-cpu_mem_load.png)
![Server Memory Statistics](https://github.com/sfuerte/zbx-mssql/blob/master/images/zbx_mssql-server_memory.png)

## System requirements

- Optional, ODBC driver installed and configured on Zabbix Server. 
For Debian 9 (stretch), install the following packages via: 
`apt-get install odbcinst tdsodbc unixodbc`


## Features

- State of MS SQL services
- Global Server Statistics (70 items in total):
  - Memory
  - Cache
  - Buffer Manager
  - Access Methods
  - Locks
  - Errors and Failed Jobs
  - Log Size
- Database discovery via Powershell (Zabbix Agent) or ODBC directly
- Statistics for discovered DB's (per DB):
  - State
  - Trasactions
  - Log File size, utilization, flushes, growths, shrinks, etc.
- 18 graphs
- Version information
- `{$NGINX_HOST}` and `{$NGINX_PORT}` macros for customization
- Tested on MS SQL 2008 & 2012. 


## Installation
 
### Zabbix Configuration

1) Copy `userparameter_mssql.conf` to your Zabbix Agent folder, e.g. `C:\srv\ZabbixAgent\conf\conf.d`.

1) Copy PowerShell scripts (`mssql_*.ps1`) to the scripts folder, e.g. `C:\srv\ZabbixAgent\scripts` folder.
IMPORTANT: if you use another folder for agent scripts, then update userparameter file in the previous step!

1) Import XML template file (`zbx_template_mssql.xml`) into Zabbix via Web GUI (Configuration -> Templates -> Import).
Optional, if you want to use ODBC for discovering databases, then update XML file before importing, as all item prototypes
are under a discovery rule that utilizes PowerShell via Zabbix Agent.
![Discovery Rule](https://github.com/sfuerte/zbx-mssql/blob/master/images/zbx_mssql-discovery_rules.png)

1) Configure regular expression in "Administration -> General -> Regular Expressions (dropdown on the right)":
```
Name: Databases for discovery
Expression: ^(master|model|msdb|ReportServer|ReportServerTempDB|tempdb)$
Type: Result is FALSE
```

1) Import "MS SQL Server database state" value mapping (`zbx_valuemaps_mssql.xml`) in "Administration -> General -> Value mapping (dropdown on the right)". 
Or add it manually:
```
 0 -> ONLINE
 1 -> RESTORING
 2 -> RECOVERING
 3 -> RECOVERY PENDING
 4 -> SUSPECT
 5 -> EMERGENCY
 6 -> OFFLINE
 7 -> Database Does Not Exist on Server
```

1) Assign the imported template to a host.

1) Optional, if using ODBC, follow instructions below.

1) Restart Zabbix Agent and enjoy... or, actually, good luck in tuning MS SQL ;) 


### ODBC Configuration

1) In MS SQL, create a read-only user for accessing `master` DB, e.g. `zbx-maint`.

1) On Zabbix Server, install required packages:
`apt-get install odbcinst tdsodbc unixodbc`

1) Create ODBC driver configuration file:
```
sudo vi /etc/odbc.ini
[srv01-mssql]
Driver = FreeTDS
Server = <SQL Server FQDN or IP>
PORT = 1433
TDS_Version = 8.0
```

1) Update macros for the monitored host under "Configuration -> Hosts -> <host> -> Macros":
```
{$MSSQL_PASSWORD} - <your generated pass>
{$MSSQL_USER}	  - zbx-maint
{$ODBC}		  - srv01-mssql
```

!Host Macros](https://github.com/sfuerte/zbx-mssql/blob/master/images/zbx_mssql-macros_config.png)


## Troubleshooting

Check ODBC settings and credentials from console on Zabbix Server:

```shell
> isql -v srv01-mssql zbx-maint <password>
+---------------------------------------+
| Connected!                            |
|                                       |
| sql-statement                         |
| help [tablename]                      |
| quit                                  |
|                                       |
+---------------------------------------+
SQL>
```
