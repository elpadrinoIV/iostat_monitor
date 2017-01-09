# Introduction
This simple program monitors io usage using the `iostat` command and makes them available via SNMP.

The idea was taken from https://github.com/markround/Cacti-iostat-templates

# Usage
## Build
```bash
go build
```

## SNMPD configuration
`iostat_monitors` connects to the snmp daemon via agentX. Snmpd configuration file must have agentX enabled:
```conf
# minimal /etc/snmp/snmpd.conf file
agentAddress udp:161,udp6:[::1]:161

rocommunity public

master          agentx
agentXSocket    tcp:localhost:705
```

## Run
The program uses the `iostat` utility, therefore `sysstat` must be installed.
```bash
./iostat_monitor
```

## Query via snmp
`iostat_monitor` registers the oid `1.3.6.1.3.1` as base oid.

OID | Meaning
-----|--------
BASE.1.DEVICEIDX | Index
BASE.2.DEVICEIDX | Device Name
BASE.3.DEVICEIDX | rrqm/s
BASE.4.DEVICEIDX | wrqm/s
BASE.5.DEVICEIDX | r/s
BASE.6.DEVICEIDX | w/s
BASE.7.DEVICEIDX | rkB/s
BASE.8.DEVICEIDX | wkB/s
BASE.9.DEVICEIDX | avgrq-sz
BASE.10.DEVICEIDX | avgqu-sz
BASE.11.DEVICEIDX | await
BASE.12.DEVICEIDX | r_await
BASE.13.DEVICEIDX | w_await
BASE.14.DEVICEIDX | svctm
BASE.15.DEVICEIDX | %util

```bash
$ snmpget -v2c -Ofn -c public 127.0.0.1 .1.3.6.1.3.1.2.1
.1.3.6.1.3.1.2.1 = STRING: "dm-0"
$ snmpget -v2c -Ofn -c public 127.0.0.1 .1.3.6.1.3.1.2.2
.1.3.6.1.3.1.2.2 = STRING: "dm-1"
$ snmpget -v2c -Ofn -c public 127.0.0.1 .1.3.6.1.3.1.2.7
.1.3.6.1.3.1.2.7 = STRING: "sda"

# Get %util of "sda"
$ snmpget -v2c -Ofn -c public 127.0.0.1 .1.3.6.1.3.1.15.7
.1.3.6.1.3.1.15.7 = STRING: "0.00"
```

# Known issues
Currently supports only `iostat` version >= 10 since older versions of `iostat` don't give `r_await` nor `w_await`:
```bash
$ iostat -V
sysstat version 10.1.5
(C) Sebastien Godard (sysstat <at> orange.fr)
$ iostat -xk
Device:         rrqm/s   wrqm/s     r/s     w/s    rkB/s    wkB/s avgrq-sz avgqu-sz   await r_await w_await  svctm  %util
sda               0.00     0.00    0.00    0.00     0.00     0.00     0.00     0.00    0.00    0.00    0.00   0.00   0.00

$ iostat -V
sysstat version 7.0.2
(C) Sebastien Godard
$ iostat -xk
Device:         rrqm/s   wrqm/s   r/s   w/s    rkB/s    wkB/s avgrq-sz avgqu-sz   await  svctm  %util
sda               0.00     0.00  0.00  0.00     0.00     0.00     0.00     0.00    0.00   0.00   0.00
```

