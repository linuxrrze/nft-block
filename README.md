# nft-block
Script for blocking IPs in nftables using blacklists.

## Features
* Simple, powerful and easy to use
* Purely written in Bash with no dependency on Python
* Daemon-less
* Does not interfere with user-defined nftables rules

## Requirements
* nftables
* bash
* curl

## Description
__nft-block__ is a simple Bash script for applying and updating IP blacklists from remote blacklist files.\
All input/output traffic from/to blacklisted IPs will be blocked.

__nft-block__ _does not_ interfere with user-defined nftables rules.
* Rules created by this script are stored in `nft-block` table.
* Created filter chains have a high priority of -190 (most user-defined filter chains do not use such high priority) 

While installation is _not necessary_, without installation applied rules will not persist between system startups and won't be kept up-to-date.

## Usage
```
$ # Clone this project
$ git clone https://github.com/me-asri/nft-block
$ cd nft-block
$ # Execute nft-block as root
# ./nft-block -h
Usage: nft-block <options>
 Example:
  - Apply blacklist: nft-block -l https://feodotracker.abuse.ch/downloads/ipblocklist_recommended.txt
  - Install:         nft-block -i -l https://feodotracker.abuse.ch/downloads/ipblocklist_recommended.txt
  - Uninstall:       nft-block -u
 
 Options:
   -l blacklist                : Apply blacklist
                                 - Can be specified multiple times to apply as many blacklists as needed
   -x [protocol://]host[:port] : Use specified proxy
   -r                          : Clear nft-block firewall rules
   -i                          : Install to system
   -u                          : Uninstall from system
```

## Example
### Applying Feodo Tracker Botnet C2 IP blacklist
Applying a blacklist from a remote source is as simple as:
```
# ./nft-block -l https://feodotracker.abuse.ch/downloads/ipblocklist_recommended.txt
[*] Adding firewall rules
[*] Loading blacklist https://feodotracker.abuse.ch/downloads/ipblocklist_recommended.txt
[*] Adding 168 IPv4 addresses to blacklist
[*] Done!
```
### Applying Feodo Tracker Botnet C2 IP blacklist and blocklist.de IP blacklist
It's also possible to apply multiple blacklists at the same time:
```
# ./nft-block -l https://feodotracker.abuse.ch/downloads/ipblocklist_recommended.txt \
              -l https://lists.blocklist.de/lists/all.txt
[*] Adding firewall rules
[*] Loading blacklist https://feodotracker.abuse.ch/downloads/ipblocklist_recommended.txt
[*] Loading blacklist https://lists.blocklist.de/lists/all.txt
[*] Adding 16276 IPv4 addresses to blacklist
[*] Adding 21 IPv6 addresses to blacklist
[*] Done!
```
### Clearing nft-block rules
Clearing rules applied by nft-block is as simple as:
```
# ./nft-block -r
[*] Clearing nft-block firewall rules
```

## Installation & Persistence
It's possible to install __nft-block__ on your system to keep blocked IPs up-to-date and achieve persistence.
> __nft-block__ supprots persistence only on Systemd systems using timers.\
> For systemd-less systems a cron job must be manually inserted.  

### Installation
Passing in the `-i` option while executing __nft-block__ will
* install __nft-block__ to `/usr/local/bin/nft-block`
* create a persistence service `nft-block.service` and timer `nft-block.timer` that execute __nft-block__ with the same arguments used _every 5 minutes_ to update rules.

#### Example
```
# ./nft-block -i -l https://feodotracker.abuse.ch/downloads/ipblocklist_recommended.txt
[*] Installing nft-block
[*] Installed nft-block to /usr/local/bin/nft-block
[*] Using systemd timer for persistence
[*] Creating nft-block service
[*] Creating nft-block timer
Created symlink /etc/systemd/system/timers.target.wants/nft-block.timer → /etc/systemd/system/nft-block.timer.
[*] Adding firewall rules
[*] Loading blacklist https://feodotracker.abuse.ch/downloads/ipblocklist_recommended.txt
[*] Adding 168 IPv4 addresses to blacklist
[*] Done!
```

### Uninstallation
The `-u` option will uninstall __nft-block__ from the system along with installed persistence service and remove any firewall rules applied.
```
# nft-block -u
[*] Uninstalling nft-block
[*] Removing nft-block timer
Removed "/etc/systemd/system/timers.target.wants/nft-block.timer".
[*] Removing nft-block service
[*] Removing /usr/local/bin/nft-block
[*] Removing nft-block firewall rules
```

### Managing persistence
The persistence service and timer can be controlled using `systemctl`

#### Stopping timer
```
# systemctl stop nft-block.timer
```
#### Starting timer
```
# systemctl start nft-block.timer
```
#### Disabling timer on system startup
```
# systemctl disable nft-block.timer
```
#### Enabling timer on system startup
```
# systemctl enable nft-block.timer
```
#### Checking last status
```
# systemctl status nft-block.service
```

## Blacklists
Some IP blacklists to use with __nft-block__:
* [Feodo Tracker](https://feodotracker.abuse.ch/blocklist/) - Botnet C2 IP blacklist
* [blocklist.de](https://www.blocklist.de/en/index.html) - fail2ban IP blacklist

Many more blacklists can be found on the [FireHOL IP Lists](https://iplists.firehol.org/) website.

## Disclaimer
This software is provided with absolutely no warranty.\
Use at your own risk.