# Misc
forked from main branch and integrated modification from 
https://github.com/Wolvverine/zabbix-ibm-storwize

# changes
this version works on 
    - debian 11
    - python 3.9.2
    - Zabbix 5.0 LTS

# zabbix-ibm-storwize
Python script for monitoring IBM Storwize storages


# Setup
## changes
I've created a folder for it in /tmp called zbx
change it to your hearts content on line 157 in script "/tmp/zbx"
## basics
For my use case i had to install zabbix-proxy-"dbtype" zabbix-agent zabbix-sender
This is how i set it up:
 - Connected zabbix proxy to server
 - Connected zabbix agent to proxy (used server 127.0.0.1)
 - Set the zabbix server host to use a proxy
 - Host interface is "Agent" type and uses the storage IP

In template "Template EMC Unity REST-API" in section "Macros" need set these macros:
- {$STORWIZE_USER}
- {$STORWIZE_PASSWORD}
- {$STORWIZE_PORT} (22)

## the real magic

In agent configuration file, **/etc/zabbix/zabbix_agentd.conf** must be set parameter **ServerActive=xxx.xxx.xxx.xxx**

- In Linux-console need run this command to make discovery. Script must return value 0 in case of success.
```bash
./storwize_get_state.py  --storwize_ip=xxx.xx.xx.xxx --storwize_port=22 --storwize_user=user_name_of_storagedevice --storwize_password='password' --storage_name="storage_name_in_zabbix-web-interface" --discovery
```

Value of  --storage_name  is macros value {HOST.NAME}


- On zabbix proxy or on zabbix servers need run **zabbix_proxy -R config_cache_reload** (zabbix_server -R config_cache_reload)

- In Linux-console need run this command to get value of metrics. Scripts must return value 0 in case of success.
```bash
./storwize_get_state.py  --storwize_ip=xxx.xx.xx.xxx --storwize_port=22 --storwize_user=user_name_of_storagedevice --storwize_password='password' --storage_name="storage_name_in_zabbix-web-interface" --status
```
If you have executed this script from console from user root or from another user, please check access permission on file **/tmp/storwize_state.log**. It must be allow read, write to user zabbix.

## troubleshoot-ish
- !! DO NOT FORGET TO RELOAD CACHE ON SERVER/PROXY!!
- You should run these until both --status and --discover returns 0
- If you get status 2 check the data you send to zabbix comment the line 163 in storewize_get_state.py script "os.remove(temp_file)" now you'll have a file to submit to your zabbix the command is: 
```bash
/usr/bin/zabbix_sender -vv -c /etc/zabbix/zabbix_agentd.conf -s **storagename** -T -i /tmp/zbx/**filename**.tmp
```
