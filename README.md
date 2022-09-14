# On Vmware's Market Place is a new Serice Installer with Harbor (There are some issues with Harbor and does not seem to get installed or started). 

## Here are some steps to fix the shortcoming: 


### Deploy the OVA: 

![Version](https://github.com/ogelbric/Arcas_with_Harbor/blob/main/ova1.png)

![Version](https://github.com/ogelbric/Arcas_with_Harbor/blob/main/ova2.png)

![Version](https://github.com/ogelbric/Arcas_with_Harbor/blob/main/ova3.png)

![Version](https://github.com/ogelbric/Arcas_with_Harbor/blob/main/ova4.png)

![Version](https://github.com/ogelbric/Arcas_with_Harbor/blob/main/ova5.png)

### Power on the deployed VM and ssh to it

### Run a few commands to see that nothing is running and we have a mount point 

```
docker images -a
docker ps -a
lsof | grep -i Harbor
netstat -na | grep -i Harbor
df -h | grep -i Harbor  
```

### And harbor wants to run but errors out

```
systemctl status harbor
journalctl -u harbor.service | tail -20
```

### Looks like there is a directory missing for Harbor

```
ls -l `grep Working /etc/systemd/system/harbor.service | awk -F '=' '{print $2}'`

ls: cannot access '/opt/vmware/arcas/tools/harbor': No such file or directory
```

So here is the root of the problem!

### Lets check out the directory below and we see the tar file is not extracted 

```
cd /opt/vmware/arcas/tools
ls
tar xvfz harbor-offline-installer-v2.5.3+vmware.1.tgz
```

### In the new directory there is a template file for Harbor that needs to be copied and modified 

```
cp harbor.yml.tmpl harbor.yml   
```

### What needs to be changed

```
# Before we change anything lets get a few falues we need, IP and mount point

root@arcas [ /opt/vmware/arcas/tools/harbor ]# nslookup `hostname` | grep Address | tail -1 | awk '{print $2}'
192.168.1.39

root@arcas [ /opt/vmware/arcas/tools/harbor ]# df -h | grep -i harbor  | awk '{print $6}'
/harbor_storage

cat harbor.yml | grep port | grep -v '#'
  port: 80
  port: 443
  
```
### Now vi the file

```
Hostname -> IP filed 
  hostname: reg.mydomain.com -> 192.168.1.39
Port Number add one
  80 -> 81
  443 -> 444
Mount Point
  data_volume: /data  -> data_volume: /harbor_storage

```




