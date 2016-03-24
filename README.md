##nmon for Linux
[![Build Status](https://travis-ci.org/axibase/nmon.svg)](https://travis-ci.org/axibase/nmon)
 

"... systems administrator, tuner, benchmark tool gives you a huge amount of important performance information in one go.", according to http://nmon.sourceforge.net/pmwiki.php

 

This fork fixes some of the issues in the original ```makefile```. 
The project also hosts binary releases for supported linux distributions: Ubuntu, Debian, RedHat, CentOS, SLES. The binaries can be downloaded from [nmon release page](https://github.com/axibase/nmon/releases)

 

[![Chartlab Portal](https://axibase.com/wp-content/uploads/2016/03/nmon.png)](https://apps.axibase.com/chartlab/ac003f06)

# Installation

## Download [binary release](https://github.com/axibase/nmon/releases) :

```
curl -OL https://github.com/axibase/nmon/releases/download/16d/nmon_x86_ubuntu134
chmod +x nmon_x86_ubuntu134
```

## Build

Download the latest version using git clone command:

```bash
git clone git://github.com/axibase/nmon.git
```

To download a specific branch use the following command:

```bash
git clone git://github.com/axibase/nmon.git -b 16d
```

Change to the nmon source directory and execute build.sh script to compile nmon.


```bash
cd nmon
./build.sh
```

Alternatively, launch 'make' utility to compile nmon from sources for your distribution:

```
make nmon_arm_raspian
```


If compilation was successful, an nmon file will be created in the current directory.

You can now launch nmon by typing ```./nmon_{yourDistribution}```. 

# Unistall
Remove nmon binary file:

```bash
rm nmon_{yourDistribution}
```

# Usage Examples

> **Note:** Make sure that ```/opt/nmon/nmon``` binary is exist and executable.

## Launch Nmon Console

```
/opt/nmon/nmon
```

## Collect Nmon Files Locally

* Add the following row to your cron schedule:

```
0 * * * * /opt/nmon/nmon -f /opt/nmon/ -s 60 -c 60 -T
```

## Upload Hourly Files to ATSD with wget


* Create a file ```/opt/nmon/nmon_script.sh``` and add the following row to your cron schedule:

```
0 * * * * /opt/nmon/nmon_script.sh
```

* Put the following code to ```/opt/nmon/nmon_script.sh```:

> **Note:** Replace ```atsd_user, atsd_password, atsd_server``` with your real credentials.

```bash
#!/bin/sh
fn="/nmon/nmon/`date +%y%m%d_%H%M`.nmon";pd="`/opt/nmon/nmon -F $fn -s 60 -c 60 -T -p`"; \
while kill -0 $pd; do sleep 15; done; \
wget -t 1 -T 10 --user=atsd_user --password=atsd_password --no-check-certificate -O - --post-file="$fn" \
--header="Content-type: text/csv" "https://atsd_server/api/v1/nmon?f=`basename $fn`"
```

## Upload Hourly Files to ATSD with UNIX Socket

> **Note:** require ```bash```

* Create a file ```/opt/nmon/nmon_script.sh``` and add the following row to your cron schedule:

```
0 * * * * /opt/nmon/nmon_script.sh
```

* Put the following code to ```/opt/nmon/nmon_script.sh```:

> **Note:** Replace ```atsd_server``` with your real server name or address.

```bash
#!/bin/bash
fn="/opt/nmon/`date +%y%m%d_%H%M`.nmon";pd="`/opt/nmon/nmon -F $fn -s 60 -c 60 -T -p`"; \
while kill -0 $pd; do sleep 15; done; \
{ echo "nmon p:default e:`hostname` f:`hostname`_file.nmon"; cat $fn; } > /dev/tcp/atsd_server/8081
```


## Upload Hourly Files to ATSD with nc 

* Create a file ```/opt/nmon/nmon_script.sh``` and add the following row to your cron schedule:

```
0 * * * * /opt/nmon/nmon_script.sh
```

* Put the following code to ```/opt/nmon/nmon_script.sh```:

> **Note:** Replace ```atsd_server``` with your real server name or address.

```bash
#!/bin/sh
fn="/opt/nmon/`date +%y%m%d_%H%M`.nmon";pd="`/opt/nmon/nmon -F $fn -s 60 -c 60 -T -p`"; \
while kill -0 $pd; do sleep 15; done; \
{ echo "nmon p:default e:`hostname` f:`hostname`_file.nmon"; cat $fn; } | nc atsd_server 8081
```


