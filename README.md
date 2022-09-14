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

