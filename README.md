# zabbix-esxi-hpssa
Zabbix template for ESXi HP SmartArray monitoring

* hpssa.conf - UserParameters for zabbix agent
* zabbixAgent.xml - firewall ruleset for ESXi

### Installation

#### Part 1. HP SSACLI installation on ESXi
Download and install HPE Utilities Offline Bundle for ESXi <your version>. You can find it on https://support.hpe.com .

1. For example, you can download hp-HPUtil-esxi6.0-bundle-2.4-5.zip for ESXi 6.0.
2. Enable SSH connection to ESXi host.
3. Copy hp-HPUtil-esxi6.0-bundle-2.4-5.zip to ESXi host using `scp`.
4. Install util-bundle with `esxcli software vib install -d "/path/to/hp-HPUtil-esxi6.0-bundle-2.4-5.zip"`. 
Reboot the host. After that you should have `esxcli hpssacli` namespace.
5. Check `esxcli hpssacli` namespace by running `esxcli hpssacli cmd -q 'version'`
It should return you installed HPSSACLI version.

Caution
> Different versions of HP offline bundle utilites are support different controllers of HP Smart Array devices. As newer controller do you have, as newer version of offline bundle utilites you should download.
> Early versions of HP offline bundle provide `hpssacli` but new one provides `ssacli`. It is the same utils but must be used for different controllers. 
> Otherwise you will get response **'No controllers detected.'**

#### Part 2. Zabbix Agent installation on ESXi

Download and install Zabbix Agent binary for ESXi. 
You have to grab Zabbix agent for **Linux 2.6** amd64 build from http://www.zabbix.com/download.php

1. For example, you can download zabbix_agents_3.2.0.linux2_6.amd64.tar.gz for ESXi 6.0 
2. Copy zabbix_agents_3.2.0.linux2_6.amd64.tar.gz to ESXi host using `scp`.
3. Create zabbix agent directory on your datastorage (for example `DATASTORAGE1`).
**ESXi root directory  `/`  is not persistent**, any changes there will be lost after reboot.
Run `mkdir /vmfs/volumes/DATASTORAGE1/.zabbix`
4. Unpack zabbix agent to created direcroty
`tar vxfz zabbix_agents_3.2.0.linux2_6.amd64.tar.gz -C /vmfs/volumes/DATASTORAGE1/.zabbix/`
5. Place (download on you local PC and then `scp` to ESXi host) `zabbixAgent.xml` from repo to zabbix agent conf directory `/vmfs/volumes/DATASTORAGE1/.zabbix/conf/`.
6. Place `hpssa.conf` from repo to zabbix agent `conf.d` directory `/vmfs/volumes/DATASTORAGE1/.zabbix/conf//zabbix_agentd`.
7. Do not forget to configure zabbix_agentd.conf to accept connections from your zabbix server.

Now add commands to ESXi boot to run zabbix agent and enable firewall rule.
Use `vi` editor to change `/etc/rc.local.d/local.sh`. This file is only persistent after ESXi reboot.
Add this lines **before** `exit 0` string.
```bash
# Run zabbix agent from datastorage
/vmfs/volumes/DATASTOREGE1/.zabbix/sbin/zabbix_agentd -c /vmfs/volumes/DATASTOREGE1/.zabbix/conf/zabbix_agentd.conf

# Enable firewall ruleset for zabbix agent
cp /vmfs/volumes/DATASTOREGE1/.zabbix/conf/zabbixAgent.xml /etc/vmware/firewall/zabbixAgent.xml
esxcli network firewall refresh
esxcli network firewall ruleset set --enabled true --ruleset-id=zabbixMonitoring
``` 

After that all operations on ESXi host are done. You can disable SSH service on it.

#### Part 3. Zabbix Server configuration

1. Donwload from repo and import to zabbix server Template ESXi HP Smart Array.
2. Add template to host configuration.