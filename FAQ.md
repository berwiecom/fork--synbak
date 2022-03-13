

ORIGINAL was a TEXT file  -  This is the new modified MARKDOWN version.


# Frequently Asked Questions about **synbak** usage
Last Update: 2014-12-12   @ugoviti  
Last Update: 2022-03-17   @berwiecom


Troubleshooting
------------

Q: **synbak** can't create HTML reports in servers with [SELinux](https://selinuxproject.org) enabled  
#### A: Before running the first backup, we must enable SELinux context against web root:  
   `semanage fcontext -a -t httpd_sys_content_t /var/www/html/admin`


Architecture
------------

### Q: I like **synbak** and I want it translated to my language, but I don't know where to begin!  
#### A: **synbak** usees the standard [GNU gettext](https://www.gnu.org/software/gettext/) localization system.  
   Translating **synbak** is very simple, use the following steps:

   1) download the **synbak** tar.gz source package
   2) `tar zxvf synbak-x.x.x.tar.gz`
   3) `cd synbak-x.y.z/po`
   4) copy synbak.pot to language.po (example: use 'de.po' for German: `cp synbak.pot de.po`)
   5) translate all (msgstr "") lines to your language and send them to opensource@initzero.it. They will be included in the next **synbak** release.

   Many thanks for supporting **synbak**.


### Q: I want to make daily, weekly and monthly backups, what is the easiest and fastest method?  
A1: You can use the *backup_schedule function* from the config file, it's very simple:
```
   backup_schedule                 = yes

   backup_schedule_daily_keep      = 5
   backup_schedule_weekly_keep     = 4
   backup_schedule_monthly_keep    = 6
   backup_schedule_yearly_keep     = 2

   backup_schedule_daily_cron      = 1,2,3,4,5
   backup_schedule_weekly_cron     = 7
   backup_schedule_monthly_cron    = 31
   backup_schedule_yearly_cron     = 12-31
```
   **synbak** will automatically detect what type of backup to run and will change the destination directory.

A2: (DEPRECATED AND TOO COMPLEX!) 
   You can use the *-o command option* to override every file configuration variable, so you can use only one config file for each system and need to change only the backup destination.  
   For example, use the following commands in the user crontab:  
```
   00 00 * * 1-6 synbak -s fileserver1 -m mysql -M split -o "backup_erase_after=6"  -o "backup_destination=/srv/backup/fileserver1/mysql/daily"
   00 00 * * 7   synbak -s fileserver1 -m mysql -M split -o "backup_erase_after=4"  -o "backup_destination=/srv/backup/fileserver1/mysql/weekly"
   00 00 1 * *   synbak -s fileserver1 -m mysql -M split -o "backup_erase_after=12" -o "backup_destination=/srv/backup/fileserver1/mysql/monthly"
   59 23 31 12 * synbak -s fileserver1 -m mysql -M split -o "backup_erase_after=5"  -o "backup_destination=/srv/backup/fileserver1/mysql/yearly"
 ```
   If using *rsync* and *backup_incremental=no* you can use the powerful rsync hard link method to make full backups using the same space of an incremental backup.  
   Across daily/weekly/monthly backups:
```
   00 00 * * 2-6 synbak -s fileserver1 -m rsync -o "backup_erase_after=6"  -o "backup_destination=/srv/backup/fileserver1/rsync/daily"
   00 00 * * 7   synbak -s fileserver1 -m rsync -o "backup_erase_after=4"  -o "backup_destination=/srv/backup/fileserver1/rsync/weekly"  -o "backup_method_opts=--link-dest=/srv/backup/fileserver1/rsync/daily/backup-fileserver1"
   00 00 1 * *   synbak -s fileserver1 -m rsync -o "backup_erase_after=12" -o "backup_destination=/srv/backup/fileserver1/rsync/monthly" -o "backup_method_opts=--link-dest=/srv/backup/fileserver1/rsync/daily/backup-fileserver1"
   59 23 31 12 * synbak -s fileserver1 -m rsync -o "backup_erase_after=10" -o "backup_destination=/srv/backup/fileserver1/rsync/yearly"  -o "backup_method_opts=--link-dest=/srv/backup/fileserver1/rsync/daily/backup-fileserver1"
```

### Q: How does the 'backup_exclude' directive work, and how can I exclude directory paths with spaces?  
#### A: The *backup_exclude directive* needs directory paths or regexp patterns (which is valid for *backup_source* also).  
   For example:  
   `backup_exclude = *.ldb /tmp /proc /sys /some/path/to/exclude`

   If you want to exclude directories with spaces in their name, you can use the escape key "\", for example:  
   `backup_exclude = /Documents\ and\ Settings/UserName /a\ path\ with\ spaces/direcory/subdirectory/another\ directory/`


### Q: I'm attempting to backup local files to a remote SMB filesystem, the FAQ shows how to do the opposite.  
   Also, can I backup from one SMV device to another SMB device? 
#### A: **synbak** can act as backup server, collecting data 'from remote' host shares 'to local' **synbak** hard disks,  
   but not 'from remote' host shares 'to remote' host shares, because you only can specify 'remote_source' and  
   not 'remote_destination': Destination is always local (ex. file system directory, tape device, laserdisc media, tar archive, etc...).

   However, you can manage that situation using the *autofs* daemon of Linux, so you can mount and umount shares of  
   remote disks on demand, just configure synbak's source and destination paths to the locally mounted shares.

   For example to backup from and to network CIFS share:  
```
   -- file: /etc/auto.master ---
   /mnt/autofs/cifs /etc/auto.cifs --timeout 180
   -- file: /etc/auto.master ---

   -- file: /etc/auto.cifs ---
   serversrc -fstype=cifs,ro,username=someuser,password=somepwd,debug=0 ://serversrc/share
   serverdst -fstype=cifs,rw,username=someuser,password=somepwd,debug=0 ://serverdst/share
   -- file: /etc/auto.cifs ---

   -- file: ~/.synbak/rsync/system.conf ---
   ...
   backup_source      = /mnt/autofs/cifs/serversrc/*
   backup_destination = /mnt/autofs/cifs/serverdst/
   ...
   -- file: ~/.synbak/rsync/system.conf --- 
```

Methods
------------

### Q: How can I see what methods are supported by **synbak**?  
#### A: Run **synbak** with:  

   `> synbak -s system -m something`

   Of course, something can be anything :-)


### Q: I want to make a backup of remote machine files to a local directory, what method can I use?  
#### A: The suggested method to backup remote machine files is 'rsync'.  
   Just comment out the 'backup_source_uri' and 'report_remote_uri_down' variables  
   and use a valid URI pointing to the remote machine.  
   **synbak**'s remote rsync method is born to supply an universal way to backup files to a local directory, supporting many ways to achieve that.  
   For example:

   ## rsync via ssh daemon (useful to make secure backup via untrusted network like internet... suggested method)
   1) assume that remote source host name or ip address to backup is for example: 'www.example.com'
   2) if you want backup all remote system files and preserve file ownership you must launch **synbak** using the 'root' user
   3) your **synbak** server public ssh certificate is installed in the source host (/root/.ssh/authorized_keys)
   4) the ssh daemon is running in the standard ssh port (TCP-22) of the source host

   in your **synbak** system config file put:  
   `backup_source_uri = ssh://root@www.example.com`

   if your remote ssh server is running in another port (not the default 22) like port 22622, use:  
   `backup_source_uri = ssh://root@www.example.com:22622`

   if you don't want use the root account fo security reason you can use:  
   `backup_source_uri = ssh://username@www.example.com`

   TIPS to create ssh key pairs:  
   - create (if not exist) the ssh keys into 'synbak server':  
     `$ ssh-keygen -t dsa`
   - Copy the **synbak** ssh public certificate to the 'authorized_keys' file, located into source host:  
     `$ scp ~/.ssh/id_dsa.pub root@www.example.com:.ssh/authorized_keys`  
     (WARNING: the above command overwrite the remote ~/.ssh/authorized_keys if already exist)  
     or  
     cat ~/.ssh/id_dsa.pub and 'APPEND' the output to the ~/.ssh/authorized_keys of remote host.  
   - Before launch the backup procedure, try to connect from the **synbak** server to the source host with ssh command:  
     ```
	 $ ssh root@www.example.com
	 ```
   - if you can connect without password prompt, you can make backups also.

   - if you don't want enable remote root access and must backup all remote filesystem, to greatly enhance security 
   - when connecting **synbak** to remote host, you can enable the variable 'method_rsync_sudo=yes'.  
   - **synbak** will connect to the remote host and will run rsync daemon via sudo command  
   - you must add the specified user of backup_source_uri into remote server sudoers file  
   - follow this steps to make **synbak** connect as unprivileged user but backup remote file system as root users:  
   
   1) configure **synbak** with:  
      ```
	  backup_source_uri=ssh://synbak@www.example.com:22622
      method_rsync_sudo=yes
	  ```
   2) ssh into remote server as root and create a new unprivileged user named 'synbak'
      (you can use whatever you want of course) run: useradd synbak
   4) run: `visudo`
      and add to the end of file:
      ```
	  Defaults:synbak !requiretty
      **synbak** ALL=(root) NOPASSWD: /usr/bin/rsync
	  ```
 

   ## rsync via rsync daemon (unsuggested because the connection is unencrypted)
   1) assume that remote host name or ip address is: 192.168.0.1
   2) in the remote machine is running a plain rsync daemon (port 873 open)
   3) the remote rsync share is named: backup
   4) you are able to connect to remote rsync daemon from your **synbak** server
   
   in your **synbak** system config file you must put:
   `backup_source_uri = rsync://192.168.0.1/backup`


   ## rsync of a Windows machine (NetBios protocol = smb or cifs) having a standard windows share enabled
   1) assume that remote host name or ip address is: intranetserver
   2) the remote share is named backup$
   3) the remote machine and the local **synbak** server support the CIFS protocol (samba >= 3.x and kernel >= 2.6.x)
   
   In your **synbak** system config file you must put:
   `backup_source_uri = cifs://Administrator:password@intranetserver/backup$`

   If your **synbak** server doesn't support cifs (aka mount.cifs command), you can use smb protocol:
   `backup_source_uri = smb://Administrator:password@intranetserver/backup$`

   Note: If you want to maintain remote file ownership YOU MUST RUN SYNBAK as user **root**.

  

### Q: I just use **synbak**'s rsync method to make my server's data secure, but, some servers are running live databases like MySQL, Oracle or LDAP directories. So how can I make a safe and clean database dump using synbak?
#### A: Comment out 'backup_source_uri' and 'report_remote_uri_down' and use a URI like these:

   # For MySQL dumps
   backup_source_uri = mysql://username:password@hostname

   # For Oracle dumps
   backup_source_uri = oracle://username:password@SID

   # For LDAP dumps
   backup_source_uri = ldap://cn=Manager:password@hostname

   If the remote daemons are not running on the default port, you can specify the port like other URIs:
   backup_source_uri = mysql://username:password@hostname:3307

   By default, **synbak** makes a single bzip2 compressed file dump from all databases, but for the MySQL method exists a special switch that allows you to make a splitted database dump for each database found (useful for easy single database recovery).
   Launch **synbak** with the following command line:

   `> synbak -s system -m mysql -M split`


### Q: How can I know if a backup method needs or allow some extra options?
#### A: Run **synbak** with:

   `> synbak -s system -m tar -M something`

   The output will be:

   ERROR: The specified extra option 'something' is invalid, follow a list of usable extra options: tar, gz, bz2


### Q: What devices are supported by the tape method?
#### A: All devices supported by your GNU/Linux machine :-)
   You can verify the tape device name with following command:

   `$ dmesg | grep ^st`

   For example my output is:
```
   st: Version 20050830, fixed bufsize 32768, s/g segs 256
   st 5:0:5:0: Attached scsi tape st0
   st0: try direct i/o: yes (alignment 512 B)
   st0: Block limits 1 - 16777215 bytes
```
   Now you can configure a tape method using the '/dev/st0' device as backup_destination:

   `backup_destination = /dev/st0`


### Q: How can I use a multi loader tape?
#### A: Firt you must discover your tape loader SCSI device address name (generally /dev/sgX, where X is a number).
   Then you must comment out the following line:
   `backup_device_changer = /dev/sg0`

   and run a tape backup supplying the extra options:

   `synbak -s myserver -m tape -M 1`

   where 1 is the first tape number loaded in the tape changer.

   Note: If your GNU/Linux system uses the udev daemon, discovering the right device node is pretty simple: Normally it's named */dev/changer*


### Q: Can I make a full remote HTTP or FTP server mirror?
#### A: Yes, you can use the 'wget' method.
   The usage is similar to the rsync method.
   You must provide only a correct URI:
   `backup_source_uri = http://www.example.com`
   or
   `backup_source_uri = ftp://user:password@ftp.example.com`

   you can use the wget method to save a set of files also:

   `backup_source_uri = http://www.example.com/some/dir/`
   `backup_source = file1.tar.bz2 file2.odt file3.zip`

   this will save:
   http://www.example.com/some/dir/file1.tar.bz2
   http://www.example.com/some/dir/file2.odt
   http://www.example.com/some/dir/file3.zip

   remember: if you want make a full http/ftp mirror, backup_source must be empty
   or contain the special wildacard '*'


### Q: How can I use **synbak** automatically without human presence?
#### A: Unix crontab is your best friend :-)

   Launch from command prompt: `crontab -e`

   and insert something like this:
```
   00 00 * * *   synbak -s fsserver        -m ldap

   10 00 * * *   synbak -s dbserver1       -m oracle

   15 00 * * *   synbak -s dbserver2       -m mysql
   30 00 * * *   synbak -s dbserver3       -m mysql
   45 00 * * *   synbak -s dbserver4       -m mysql

   45 00 * * *   synbak -s fileserver1     -m rsync
   30 01 * * *   synbak -s fileserver2     -m rsync
   00 02 * * *   synbak -s webserver       -m rsync
   00 03 * * *   synbak -s erpserver       -m rsync
   00 04 * * *   synbak -s hostingserver   -m rsync
   00 05 * * *   synbak -s firewall        -m rsync

   00 06 * * 2   synbak -s servers         -m tape -M 1
   00 06 * * 3   synbak -s servers         -m tape -M 2
   00 06 * * 4   synbak -s servers         -m tape -M 3
   00 06 * * 5   synbak -s servers         -m tape -M 4
   00 06 * * 6   synbak -s servers         -m tape -M 5

   00 11 * * 1-5 synbak -s customerserver  -m rsync
   23 */4 * * *  synbak -s laptop          -m rsync
```

Reports
------------

### Q: How can I totally rebuild my html indexes (all days, months and years)?
#### A: Normally when you run synbak, it update the current day only, but, if you need it,
   you can rebuild your html indexes and directory tree adding the '-r html -R ria' options to your backup job.
   example:
   if your system is named 'webserver' and your method is 'ldap', you must launch **synbak** with the following command:

   > synbak -s webserver -m ldap -r html -R ria

   you should see the following output:

   Rebuilding indexes:  Generating report: 'html' Duration:[00:19:23] Status:[OK]

   now if you go to your **synbak** web reports interface you will see all indexes updated


### Q: How can I integrate **synbak** reports with network monitoring software like Nagios or Icinga?
#### A: Since 3.1 release, **synbak** implement ad rss feed report about overall backup reports status.
   Synbak, after every backup job, update the file 'status.xml' located into 'report_html_uri' path
   This file contain the overall status of daily backups, counting the backups made, the successful backups, and the failed backups.
   You can integrate Synbak into nagios/icinga using the last version of 'check_rss' plugin, downloadable from:
   http://john.wesorick.com/2011/10/nagios-plugin-checkrss.html
   (I suggest to use the most updated fork from https://github.com/denisbr/nagios-plugins/blob/master/check_rss)
   and create the nagios configs like this:

```
   -- file hostgroups.cfg ---------------------
   define hostgroup{
        hostgroup_name  synbak-servers
        alias           Synbak Backup Servers
   }
   ------------------------------------------

   -- file commands.cfg ---------------------
   define command{
     command_name  check_synbak
     command_line  /etc/nagios/conf.d/plugins/check_rss -H $HOSTADDRESS$/admin/log/backup/status.xml -p -v2 -I -U -T $ARG1$ -c "Successful:[0],Riusciti:[0]" -w ":[ERROR"
   }
   ------------------------------------------

   -- file services.cfg ---------------------
   define service{
        name                    synbak-server
        use                     generic-service
        hostgroup               synbak-servers
        normal_check_interval   60
        notification_interval   1440
        register                0
   }

   define service{
        use                 synbak-server
        service_description Synbak
        check_command       check_synbak!24
   }
   -----------------------------------------
```

  24 is the time in hours after that we consider the backup status critical if none update occur


