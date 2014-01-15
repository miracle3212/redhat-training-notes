# Unit 5: Logging

### Commands
    logger
    logwatch
    logrotate

### Files
    /etc/rsyslog.conf
    /etc/logwatch/conf/logwatch.conf
    /etc/logrotate.d/

### Notes
* daemon: rsyslogd, service rsyslog restart
* facility.severity
* -/var/log/maillog - don't sync the file
* ~ - don't log, delete the message
* * - print on all terminals
* ```logger -p mail.crit "test of mail log"``` goes to /var/log/maillog, /var/log/crit
* ```logger -p cron.emerg "test of cron emerg log"```

# Unit 6: yum and rpm

## yum

### Commands
    yum 
        search
        search all
        info
        install
    	remove
    	history
    	list
    	list installed
    	list updates
    	provides
    	grouplist
    	groupinfo
    	groupinstall
    	repolist
    	clean metadata
    	clean all
    	localinstall --nogpgcheck

### Files
    /etc/yum.conf
    /etc/yum.repos.d/*.repo

### Examples
    # after removing a package, we also want to remove dependant packages
    # look for the transaction that originally installed packages (3)
    yum history 
    # get more info on that transaction
    yum history info 3 
    # undo transaction (remove packages)
    yum history undo 3
    # show which packages provides certain file or functionality
    yum provides "something" 
    # install package that provides that file
    yum install /path/to/file 

### Notes
* to create a new repo file, if we don't have any yet: man yum.conf, look for example, create a repo (e.g. updates.repo), yum list updates

## rpm

### Commands
    rpm
    repoquery
    cpio
    rpm2cpio    

### Examples
    rpm -q httpd # query package
    rpm -qi httpd # info
    rpm -ql httpd # file list
    rpm -qd httpd # documentation files
    rpm -qc httpd # configuration files
    rpm -qf /etc/httpd # which package owns files
    rpm -q --changelog httpd
    rpm -q --scripts httpd # preinstall, postinstall, preuninstall, postuninstall, pretransaction, posttransaction
    rpm -qpi /path/to/package.rpm # look into the package provided on the command line, no into database
    rpm -ivh /path/to/package.rpm # install package with rpm (not recommended)

# Notes
* midnight commander (mc) is capable of looking into rpm files

# Unit 7: SSH and Archiving

### Commands
    ssh
    scp
    rsync
    ssh-keygen
    ssh-copy-id
    tar

### Notes
* securing SSH is not on the exam

### Examples
    tar c /etc/ > etc.tar # create archive of /etc/ directory
    tar cz /etc/ > etc.tar.gz # create compressed archive of /etc/ directory using gzip
    tar czf etc.tar.gz /etc/  # same but without io redirection
    tar cjf etc.tar.bzip2 /etc/ # using bzip2 - slower but smaller archive
    tar cJfv etc.tar.xz /etc/ # using xz - best compression ratio; v - verbose
    tar tf etc.tar # "test" the archive (show files)
    tar xf etc.tar.xz # extract the archive


# Unit 8: Services

### Commands
    service X [start|stop|restart|status]
    chkconfig X [on|off]

### Files
    /etc/rc.d/init.d/

### Notes
* find rpm for service, install it, configure, start, test

## ntp

### Commands
    system-config-date
    ntpq -p

## vncserver

### Commands
    vncpasswd

### Files
    /etc/sysconfig/vncservers

### Example
    yum search vnc # reveals tigervnc-server
    yum install tigervnc-server
    rpm -qc tigervnc-server # reveals /etc/sysconfig/vncservers
    # configure vncservers (see sample config in [config/vnc](config/vnc))
    # log in as student
    su - student
    # set password
    vncpasswd
    # logout back to root
    service vncserver restart
    # test it on client machine
    vncviewer -via student@server2 localhost:1

## vsftpd

### Commands
    lftp

### Files
    /etc/vsftpd/vsftpd.conf
    /var/ftp/

### Example
    yum install vsftpd
    service vsftpd restart
    chkconfig vsftpd on
    cp /etc/resolv.conf /var/ftp/pub/
    # test it on client machine
    lftp <ipaddr>
    # or in browser: ftp://<ipaddr> or ftp://<user>@<ipaddr>

## httpd

### Files
    /etc/httpd/conf/httpd.conf
    /var/www/html/

### Example
    yum install httpd
    chkconfig httpd on
    service httpd restart
    # create index.html in /var/www/html
    # test it on client machine in browser

## iptables

### Commands
    system-config-firewall
        enable/disable, check services, apply

### Notes
* firewall configuration will not be on the exam, but firewall may be running. it is acceptable to disable it.

# Unit 9: SELinux

### Commands
    restorecon
    setsebool
    getenforce
    setenforce
    seinfo
    semanage
    setroubleshoot
    sealert

### Files
    /etc/selinux/config
    /var/log/audit/audit.log

### Notes
* NOT on the exam, but it is on the RHCE exam
* to see denials: grep AVC /var/log/audit/audit.log
* discrete access control - based on user/group membership
* problem with that approach: any app ran by user has access to all files owned by user (e.g. ssh keys)
* selinux - mandatory access control
* based on labels and policies
* to find documentation: man -k _selinux (man httpd_selinux)
* all file types are defined in a policy
* files get type from parent directories
* except for files in user's home directory
* the process will get the type from the application that is running it
* except for daemons

<pre>
4 types of access control in selinux:
1)RBAC
    user:role
    user_u:user_r
    staff_u:staff_r,sysadm_r
    unconfined_u - not restricted by the system
    su - does not change the role
2)TE (type enforcement)
    type
    httpd_sys_content_t
    httpd_conf_t
    httpd_var_log_t	
    httpd_t
    httpd_exec_t
3)Security Level
    lower level can write but not read to upper level, unless going through trusted application
    upper level can read but not write to lower level, unless going through trusted application
    same level can read and write to each other
    used mainly in military
4)Categories
    c0, ... c1023
    supersets have access to subsets
example: user_u:user_r:httpd_sys_content_t:c0,c7
</pre>