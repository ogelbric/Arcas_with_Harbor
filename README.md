# Vmware's Market Place has a Serice Installer (1.4) with Harbor
```
https://marketplace.cloud.vmware.com/services/details/service-installer-for-vmware-tanzu-1?slug=true
```
Here is a play by play on how to make it work. 

## Deploy the OVA: 

![Version](https://github.com/ogelbric/Arcas_with_Harbor/blob/main/ova1.png)

![Version](https://github.com/ogelbric/Arcas_with_Harbor/blob/main/ova2.png)

![Version](https://github.com/ogelbric/Arcas_with_Harbor/blob/main/ova3.png)

![Version](https://github.com/ogelbric/Arcas_with_Harbor/blob/main/ova4.png)

![Version](https://github.com/ogelbric/Arcas_with_Harbor/blob/main/ova5.png)

### Power on the deployed VM and ssh to it

### Run a few commands to see that everything is running and we have a mount point. 

```
docker images -a
docker ps -a
lsof | grep -i Harbor
netstat -na | grep -i Listen | grep 9443
df -h | grep -i Harbor  
```

### And Harbor is running with out errors

```
systemctl status harbor
journalctl -u harbor.service | tail -20
```

### Connect to Harbor in a browser

```
Then goto Harbor http://192.168.3.39:9080 or get re-directed to https://192.168.3.39:9443 and log on with admin/VMware1!

```

### Then download from market place tanzu_16.tar bundle (Air gap)

![Version](https://github.com/ogelbric/Arcas_with_Harbor/blob/main/tanzu_154.png)

### Copy file to Arcas machine (my case Windows to Linux): 

```
pscp -P 22 tanzu_16.tar root@192.168.3.39:/opt/vmware/arcas/tools/.
```
### Double checked up to here.....!!!! scp seems to take a while...!!!!!


### Lets see if we can Import the bundle

```
root@arcas [ /opt/vmware/arcas/tools ]# arcas --load_tanzu_image_to_harbor
Load_Tanzu_Image: Load Tanzu Images to Harbor
Load Tanzu Images to Harbor failed {'ERROR_CODE': 500, 'msg': "[Errno 2] No such file or directory: '/harbor_storage/cert/photon-machine.local.crt' Failed Harbor image push", 'responseType': 'ERROR'}
```
### Small problem fixed by moving the cert into a different file

```
root@arcas [ /opt/vmware/arcas/tools ]# cd /harbor_storage/cert/
root@arcas [ /harbor_storage/cert ]# 
root@arcas [ /harbor_storage/cert ]# cp *.crt /harbor_storage/cert/photon-machine.local.crt
root@arcas [ /harbor_storage/cert ]# arcas --load_tanzu_image_to_harbor
Load_Tanzu_Image: Load Tanzu Images to Harbor
Load_Tanzu_Image: Load Tanzu Images to Harbor Successfully
root@arcas [ /harbor_storage/cert ]# 

```
### Outcome in Harbor 135 items loaded

![Version](https://github.com/ogelbric/Arcas_with_Harbor/blob/main/harboroutcome.png)



## The manual way: 

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
  
```
### Now vi the file

```
Hostname -> IP filed 
  hostname: reg.mydomain.com -> 192.168.1.39
Port Number add one
  80 -> 81
Mount Point
  data_volume: /data  -> data_volume: /harbor_storage

Change place in this section #'s in front unless you have a cert for ssl

# https related config                                                                                                                                     
#https:                                                                                                                                                   
  # https port for harbor, default is 443                                                                                                                 
#  port: 444                                                                                                                                               
  # The path of cert and key files for nginx                                                                                                               
#  certificate: /your/certificate/path                                                                                                                     
#  private_key: /your/private/key/path   

```

### Check you work

```
grep -e hostname -e port -e data_vol harbor.yml | grep -v "#"
hostname: 192.168.1.39
  port: 81
data_volume: /harbor_storage
```

### Now install Harbor
```
cd /opt/vmware/arcas/tools/harbor
./install.sh
```

### Outcome
```
root@arcas [ /opt/vmware/arcas/tools/harbor ]# ./install.sh

[Step 0]: checking if docker is installed ...

Note: docker version: 20.10.11

[Step 1]: checking docker-compose is installed ...

Note: docker-compose version: 1.26.2

[Step 2]: loading Harbor images ...
Loaded image: vmware.io/goharbor/notary-signer-photon:v2.5.3_vmware.1
Loaded image: vmware.io/goharbor/trivy-adapter-photon:v2.5.3_vmware.1
Loaded image: vmware.io/goharbor/harbor-core:v2.5.3_vmware.1
Loaded image: vmware.io/goharbor/harbor-registryctl:v2.5.3_vmware.1
Loaded image: vmware.io/goharbor/harbor-jobservice:v2.5.3_vmware.1
Loaded image: vmware.io/goharbor/chartmuseum-photon:v2.5.3_vmware.1
Loaded image: vmware.io/goharbor/harbor-log:v2.5.3_vmware.1
Loaded image: vmware.io/goharbor/redis-photon:v2.5.3_vmware.1
Loaded image: vmware.io/goharbor/registry-photon:v2.5.3_vmware.1
Loaded image: vmware.io/goharbor/prepare:v2.5.3_vmware.1
Loaded image: vmware.io/goharbor/harbor-portal:v2.5.3_vmware.1
Loaded image: vmware.io/goharbor/nginx-photon:v2.5.3_vmware.1
Loaded image: vmware.io/goharbor/notary-server-photon:v2.5.3_vmware.1
Loaded image: vmware.io/goharbor/harbor-db:v2.5.3_vmware.1
Loaded image: vmware.io/goharbor/harbor-exporter:v2.5.3_vmware.1


[Step 3]: preparing environment ...

[Step 4]: preparing harbor configs ...
prepare base dir is set to /opt/vmware/arcas/tools/harbor
WARNING:root:WARNING: HTTP protocol is insecure. Harbor will deprecate http protocol in the future. Please make sure to upgrade to https
Generated configuration file: /config/portal/nginx.conf
Generated configuration file: /config/log/logrotate.conf
Generated configuration file: /config/log/rsyslog_docker.conf
Generated configuration file: /config/nginx/nginx.conf
Generated configuration file: /config/core/env
Generated configuration file: /config/core/app.conf
Generated configuration file: /config/registry/config.yml
Generated configuration file: /config/registryctl/env
Generated configuration file: /config/registryctl/config.yml
Generated configuration file: /config/db/env
Generated configuration file: /config/jobservice/env
Generated configuration file: /config/jobservice/config.yml
Generated and saved secret to file: /data/secret/keys/secretkey
Successfully called func: create_root_cert
Generated configuration file: /compose_location/docker-compose.yml
Clean up the input dir



[Step 5]: starting Harbor ...
Creating network "harbor_harbor" with the default driver
Creating harbor-log ... done
Creating harbor-portal ... done
Creating redis         ... done
Creating registry      ... done
Creating registryctl   ... done
Creating harbor-db     ... done
Creating harbor-core   ... done
Creating nginx             ... done
Creating harbor-jobservice ... done
✔ ----Harbor has been installed and started successfully.----
```

### Habror is running in docker

```
root@arcas [ /opt/vmware/arcas/tools/harbor ]# docker ps -a
CONTAINER ID   IMAGE                                                   COMMAND                  CREATED         STATUS                   PORTS                                   NAMES
e946636a76c3   vmware.io/goharbor/nginx-photon:v2.5.3_vmware.1         "nginx -g 'daemon of…"   6 minutes ago   Up 6 minutes (healthy)   0.0.0.0:81->8080/tcp, :::81->8080/tcp   nginx
a48bf3620c00   vmware.io/goharbor/harbor-jobservice:v2.5.3_vmware.1    "/harbor/entrypoint.…"   6 minutes ago   Up 6 minutes (healthy)                                           harbor-jobservice
87ad6bcdb24e   vmware.io/goharbor/harbor-core:v2.5.3_vmware.1          "/harbor/entrypoint.…"   6 minutes ago   Up 6 minutes (healthy)                                           harbor-core
c38c07cfe081   vmware.io/goharbor/harbor-registryctl:v2.5.3_vmware.1   "/home/harbor/start.…"   6 minutes ago   Up 6 minutes (healthy)                                           registryctl
872f71241e59   vmware.io/goharbor/harbor-db:v2.5.3_vmware.1            "/docker-entrypoint.…"   6 minutes ago   Up 6 minutes (healthy)                                           harbor-db
e93e13477a5a   vmware.io/goharbor/registry-photon:v2.5.3_vmware.1      "/home/harbor/entryp…"   6 minutes ago   Up 6 minutes (healthy)                                           registry
3e0e185ade3c   vmware.io/goharbor/redis-photon:v2.5.3_vmware.1         "redis-server /etc/r…"   6 minutes ago   Up 6 minutes (healthy)                                           redis
8aecd876a9dd   vmware.io/goharbor/harbor-portal:v2.5.3_vmware.1        "nginx -g 'daemon of…"   6 minutes ago   Up 6 minutes (healthy)                                           harbor-portal
63daac61929c   vmware.io/goharbor/harbor-log:v2.5.3_vmware.1           "/bin/sh -c /usr/loc…"   6 minutes ago   Up 6 minutes (healthy)   127.0.0.1:1514->10514/tcp               harbor-log
root@arcas [ /opt/vmware/arcas/tools/harbor ]# docker images -a
REPOSITORY                                TAG               IMAGE ID       CREATED        SIZE
vmware.io/goharbor/harbor-exporter        v2.5.3_vmware.1   5e5c4db4025b   2 months ago   87.2MB
vmware.io/goharbor/chartmuseum-photon     v2.5.3_vmware.1   ef139f7e5902   2 months ago   148MB
vmware.io/goharbor/redis-photon           v2.5.3_vmware.1   557112156e7c   2 months ago   154MB
vmware.io/goharbor/trivy-adapter-photon   v2.5.3_vmware.1   1840ef41b15d   2 months ago   244MB
vmware.io/goharbor/notary-server-photon   v2.5.3_vmware.1   fe92db1a04e8   2 months ago   99.1MB
vmware.io/goharbor/notary-signer-photon   v2.5.3_vmware.1   5db5cc148122   2 months ago   96.6MB
vmware.io/goharbor/harbor-registryctl     v2.5.3_vmware.1   2af4f234fa92   2 months ago   136MB
vmware.io/goharbor/registry-photon        v2.5.3_vmware.1   36be3ac52388   2 months ago   77.9MB
vmware.io/goharbor/nginx-photon           v2.5.3_vmware.1   1d168ab51bda   2 months ago   44.3MB
vmware.io/goharbor/harbor-log             v2.5.3_vmware.1   aee0823eb650   2 months ago   161MB
vmware.io/goharbor/harbor-jobservice      v2.5.3_vmware.1   26d82b3b0937   2 months ago   227MB
vmware.io/goharbor/harbor-core            v2.5.3_vmware.1   0ee79be0716f   2 months ago   203MB
vmware.io/goharbor/harbor-portal          v2.5.3_vmware.1   805ebb37c371   2 months ago   52.6MB
vmware.io/goharbor/harbor-db              v2.5.3_vmware.1   519b3ed6ab4d   2 months ago   224MB
vmware.io/goharbor/prepare                v2.5.3_vmware.1   fa3936095f14   2 months ago   166MB
```

### Test the login screen

```
#password is here
grep -i admin_password /opt/vmware/arcas/tools/harbor/harbor.yml | grep -v "#" 
harbor_admin_password: Harbor12345
```

![Version](https://github.com/ogelbric/Arcas_with_Harbor/blob/main/login1.png)

![Version](https://github.com/ogelbric/Arcas_with_Harbor/blob/main/login2.png)




### older trouble shooting commands


### Looks like there is a directory missing for Harbor

```
ls -l `grep Working /etc/systemd/system/harbor.service | awk -F '=' '{print $2}'`

ls: cannot access '/opt/vmware/arcas/tools/harbor': No such file or directory
```
So here is the root of the problem!

## There are 2 options - 1 a script - 2 the manual way

### The script (looks like was written for DHCP but with a teak works for static IP)

```
chmod +x /opt/vmware/arcas/bin/extract_and_install_harbor_dhcp.sh
vi /opt/vmware/arcas/bin/extract_and_install_harbor_dhcp.sh

Change from:
echo "Update admin password."
sed -i "s/harbor_admin_password: Harbor12345/harbor_admin_password: $(/opt/vmware/bin/ovfenv --key sivt.password)/g" harbor.yml
echo "Update password for the root user of Harbor DB."
sed -i "s/password: root123/password: $(/opt/vmware/bin/ovfenv --key sivt.password)/g" harbor.yml


Change to  $(cat /etc/.secrets/root_password): 
echo "Update admin password."
sed -i "s/harbor_admin_password: Harbor12345/harbor_admin_password: $(cat /etc/.secrets/root_password)/g" harbor.yml
echo "Update password for the root user of Harbor DB."
sed -i "s/password: root123/password: $(cat /etc/.secrets/root_password)/g" harbor.yml

#then run
/opt/vmware/arcas/bin/extract_and_install_harbor_dhcp.sh
```

