# Unit 1: Networking

### Commands
    ip addr show (ip a)
    ip route show (ip r)
    service NetworkManager restart
    ifdown eth0 && ifup eth0
    hostname
    host
    dig
    ping
    traceroute

### Files
    /etc/resolv.conf
    /etc/sysconfig/network-scripts/ifcfg-eth0
    /etc/sysconfig/network
    /etc/hosts
    sysconfig.txt

### Troubleshooting
Make sure you can ping the gateway:

    ping 192.168.0.254
    
Make sure DNS works:

    traceroute 8.8.8.8
    dig google.com
    host google.com

### Notes
* never change resolv.conf manually, it is overwritten every time network service is restarted; use ifcfg-<interface> to configure name servers

# Unit 2: Users and Groups

## System Users and Groups

### Commands
    system-config-users
    useradd
    groupadd
    id
    userdel -r # -r makes sure homedir & other files owned by user get deleted
    passwd # to change user password
    chage # to change age info

### Files
    /etc/passwd
    /etc/shells
    /sbin/nologin
    /etc/shadow
    /etc/group

* to force user to change password on next login, set last pwd change date to 0 with ```chage -d```
* to lock user account: ```usermod -L elvis```
* to unlock: ```usermod -U elvis```
* UIDs: ```0``` - root, ```< 500``` - system users, ```>= 500``` - regular users

## LDAP and Automount
* protocols: ```ldap://```, ```ldaps://```
* ```ldap://instructor.example.com```
* daemon: ```sldapd```
* SSL, TLS, CA_CERT, PKI

### Commands
    system-config-authentication
    service autofs stop && service autofs start # don't user restart - does not pick up new config
    yum install sssd # if not already installed
    getent passwd ldapuser1 # should print info about ldapuser1

### Files
    /etc/auto.master
    /etc/auto.guests

* see sample config in [config/automount](config/automount)

# Unit 3: Processes

### Commands
    ssh -X
    ^Z, ^C
    bg, fg
    jobs
    <command> &
    ps aux
    kill <pid>
    killall <name>
    top
    nice
    renice

* default signal sent with ```kill``` is ```15 (SIGTERM)``` which asks process nicely to die
* if we want to send a different signal: ```kill -9 <pid```, sends signal ```9 (SIGKILL)``` - kernel kill the process
* ```jobs``` to list processes, ```kill %2``` to kill proces ```[2]```
* ```jobs```: ```+``` means that process will get the signal. ```-``` means that process will next get the ```+```
* process priorities: from 1 (highest) to 39, can be viewed with top

## Cron

* to schedule tasks

### Files
    /etc/cron.daily, ...
    /var/spool/cron

### Commands
    crontab
