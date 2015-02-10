# Instructions for A Vanilla Install Unidata LDM on Ubuntu
## Request Access to disconbb16.cloudapp.net
* Find your external IP address:


```
ifconfig | sed -En 's/127.0.0.1//;s/.*inet (addr:)?(([0-9]*\.){3}[0-9]*).*/\2/p'
```
* Send an email with your IP address to  _kevin.smith@tufts.edu_

## (as superuser) Add User ldm
```
groupadd ldm
useradd -m -g users -G ldm -s /bin/bash ldm
passwd ldm
```

## (as superuser) Install pax
```
apt-get install pax
```

## Login as ldm, switch to /home/ldm
```
su - ldm
cd
```

## Download and Unpack Unidata LDM
```
wget ftp://ftp.unidata.ucar.edu/pub/ldm/ldm-6.12.6.tar.gz
gunzip -c ldm-6.12.6.tar.gz | pax -r '-s:/:/src/:'
cd ldm-6.12.6/src
```

## Configure & Make
```
./configure >&configure.log
```
 __Be sure to check _configure.log_ for errors.__

```
make install >&install.log
```
__Be sure to check _install.log_ for errors.__

## Edit /home/ldm/etc/registry.xml
_e.g._
```
vim /home/ldm/etc/registry.xml
```

#### Edit Hostname
* Make sure the host name is fully qualified.

_e.g._
```
<hostname>nwisnfie-b.cloudapp.net</hostname>
```
#### Edit Queue
* Change _path_ to the desired location. 
* Change _size_ if necessary. 

_e.g._
```
<queue>
	<path>/data/queue/ldm.pq</path>
	<size>1G</size>
	<slots>default</slots>
</queue>
```

## Edit /home/ldm/etc/ldmd.conf
_e.g._
```
vim /home/ldm/etc/ldmd.conf
```

Add the following entry to _ldmd.conf_:

```
REQUEST EXP “.*” disconbb16.cloudapp.net
```

## (optional) Edit /home/ldm/etc/pqact.conf
If you'd like incoming files written to disk, edit _pqact.conf_.

_e.g._
```
vim /home/ldm/etc/pqact.conf
```

Add the following entries to _pqact.conf_:

```
ANY     ((.*)([0-9]{4}-[0-9]{2}-[0-9]{2})(.*.nc))
        EXEC    /bin/mkdir      -p      /data/queue/\3

ANY     ((.*)([0-9]{4}-[0-9]{2}-[0-9]{2})(.*.nc))
        FILE    -close  /data/queue/\3/\1
```
_N.B._ This will create daily subdirectories at  _/data/queue/_.

* The regular expression back-references are:
	* __\1__ e.g. _nfie_hydro_region_num_18_2015-02-09.nc_  
	* __\3__ e.g. _2015-02-09_
	* so in this example _/data/queue/\3/\1_ evaluates to _/data/queue/2015-02-09/nfie_hydro_region_num_18_2015-02-09.nc_ 

## Build Product Queue
```
cd
/home/ldm/bin/ldmadmin mkqueue
```

## Launch LDM
```
/home/ldm/bin/ldmadmin start
```

## (optional, as superuser) Set to Start on Boot
_e.g._
```
vim /etc/rc.local
```
Add the following entry:
```
/home/ldm/bin/ldmadmin start > /tmp/ldmd-start.log 2>&1
```

# Example: Add A File to the Product Queue
```
/home/ldm/bin/pqinsert -v nfie_hydro_region_num_02_2015-01-27.nc
```
