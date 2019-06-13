
Question:-1   Install and configure apache/httpd

Step 1 — Installing Apache  

Before installing the apache/httpd we have to set up the hostname of the machine. under the /etc/hostname and reslove the same name
with /etc/hosts file now the hostname will ping with ip address and hostname


[root@ninjaopstree ~]# yum install httpd*

Step.1 (A)— Checking Web Server Service

[root@ninjaopstree ~]# systemctl start httpd

[root@ninjaopstree ~]# systemctl status httpd
● httpd.service - The Apache HTTP Server
   Loaded: loaded (/usr/lib/systemd/system/httpd.service; disabled; vendor preset: disabled)
   Active: active (running) since Wed 2019-06-12 16:47:43 IST; 17s ago
     Docs: man:httpd(8)
           man:apachectl(8)
 Main PID: 3749 (httpd)
   Status: "Total requests: 0; Current requests/sec: 0; Current traffic:   0 B/sec"
   CGroup: /system.slice/httpd.service
           ├─3749 /usr/sbin/httpd -DFOREGROUND
           ├─3750 /usr/sbin/httpd -DFOREGROUND
           ├─3751 /usr/sbin/httpd -DFOREGROUND
           ├─3752 /usr/sbin/httpd -DFOREGROUND
           ├─3753 /usr/sbin/httpd -DFOREGROUND
           └─3754 /usr/sbin/httpd -DFOREGROUND

Jun 12 16:47:43 ninjaopstree.com systemd[1]: Starting The Apache HTTP Server...
Jun 12 16:47:43 ninjaopstree.com systemd[1]: Started The Apache HTTP Server


[root@ninjaopstree ~]# systemctl enable httpd

[root@ninjaopstree ~]# systemctl enable httpd
Created symlink from /etc/systemd/system/multi-user.target.wants/httpd.service to /usr/lib/systemd/system/httpd.service.


Allow Apache Through the Firewall

[root@ninjaopstree ~]# firewall-cmd --permanent --add-port=80/tcp
success

[root@ninjaopstree ~]# firewall-cmd --reload
success

#########################################################################################################################


Step 2  Now I m changing the httpd port number from 80 to 8080 beacuse httpd and nginx webserver bydefault run on port number 80
        To avoid the conjecution error .
		
[root@ninjaopstree ~]# vim /etc/httpd/conf/httpd.conf

under the httpd.conf file I have change the Listen Port from 80 to 8080

Than i have allow the port to firewall 

[root@ninjaopstree ~]# firewall-cmd --permanent --add-port=8080/tcp
success

[root@ninjaopstree ~]# firewall-cmd --reload
success



#########################################################################################################################

Install and configure nginx - configure it to run as reverse proxy to apache

[root@ninjaopstree ~]# yum install nginx

[root@ninjaopstree ~]# systemctl start nginx.service

[root@ninjaopstree ~]# systemctl status nginx.service
● nginx.service - The nginx HTTP and reverse proxy server
   Loaded: loaded (/usr/lib/systemd/system/nginx.service; disabled; vendor preset: disabled)
   Active: active (running) since Wed 2019-06-12 17:30:41 IST; 16s ago
  Process: 4366 ExecStart=/usr/sbin/nginx (code=exited, status=0/SUCCESS)
  Process: 4364 ExecStartPre=/usr/sbin/nginx -t (code=exited, status=0/SUCCESS)
  Process: 4363 ExecStartPre=/usr/bin/rm -f /run/nginx.pid (code=exited, status=0/SUCCESS)
 Main PID: 4368 (nginx)
   CGroup: /system.slice/nginx.service
           ├─4368 nginx: master process /usr/sbin/nginx
           └─4369 nginx: worker process

Jun 12 17:30:41 ninjaopstree.com systemd[1]: Starting The nginx HTTP and reverse proxy server...
Jun 12 17:30:41 ninjaopstree.com nginx[4364]: nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
Jun 12 17:30:41 ninjaopstree.com nginx[4364]: nginx: configuration file /etc/nginx/nginx.conf test is successful
Jun 12 17:30:41 ninjaopstree.com systemd[1]: Failed to read PID from file /run/nginx.pid: Invalid argument
Jun 12 17:30:41 ninjaopstree.com systemd[1]: Started The nginx HTTP and reverse proxy server.
[root@ninjaopstree ~]#

Now checked the httpd/naginx service runing on which port 

[root@ninjaopstree ~]#  netstat -tulpn | grep :80
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      4368/nginx: master
tcp6       0      0 :::8080                 :::*                    LISTEN      4315/httpd


[root@ninjaopstree ~]# systemctl enable nginx.service
Created symlink from /etc/systemd/system/multi-user.target.wants/nginx.service to /usr/lib/systemd/system/nginx.service.

Now configured Nginx as a reverse proxy for Apache on CentOS

[root@ninjaopstree ~]# vim /etc/nginx/nginx.conf


server {
        listen   80;

        root /usr/share/nginx/html/;
        index index.php index.html index.htm;

        server_name _;

        location / {

        proxy_set_header X-Real-IP  $remote_addr;
        proxy_set_header X-Forwarded-For $remote_addr;
        proxy_set_header Host $host;
        proxy_pass http://172.30.24.107:8080/;
        }
}

save and closed the file & checked the the url it's working fine



#############################################################################################

Install and configure 'ntp' - with singapore time zone




[root@ninjaopstree ~]# yum install -y ntp


Then, Open the /etc/ntp.conf file and add the comments of Public Servers from pool.ntp.org (Defaults list) and 
replace it with the list provided for singapore country like as mentioned below.

[root@ninjaopstree ~]# vim /etc/ntp.conf

server 0.sg.pool.ntp.org iburst
server 1.sg.pool.ntp.org iburst
server 2.sg.pool.ntp.org iburst
server 3.sg.pool.ntp.org iburst


The Port of ntp service is 123/UDP. It is work on the transport layer. To allow the port on Centos7/RHEL7/Fedora22, following command used.

[root@ninjaopstree ~]# firewall-cmd --add-service=ntp --permanent
success

[root@ninjaopstree ~]# firewall-cmd --reload
success

[root@ninjaopstree ~]# systemctl start ntpd

[root@ninjaopstree ~]# systemctl enable ntpd


[root@ninjaopstree ~]# systemctl status ntpd
● ntpd.service - Network Time Service
   Loaded: loaded (/usr/lib/systemd/system/ntpd.service; disabled; vendor preset: disabled)
   Active: active (running) since Wed 2019-06-12 20:55:07 IST; 17s ago
  Process: 4634 ExecStart=/usr/sbin/ntpd -u ntp:ntp $OPTIONS (code=exited, status=0/SUCCESS)
 Main PID: 4635 (ntpd)
   CGroup: /system.slice/ntpd.service
           └─4635 /usr/sbin/ntpd -u ntp:ntp -g

Jun 12 20:55:07 ninjaopstree.com ntpd[4635]: Listen normally on 4 lo ::1 UDP 123
Jun 12 20:55:07 ninjaopstree.com ntpd[4635]: Listen normally on 5 enp0s3 fe80::a00:27ff:fe24:4e76 UDP 123
Jun 12 20:55:07 ninjaopstree.com ntpd[4635]: Listening on routing socket on fd #22 for interface updates
Jun 12 20:55:07 ninjaopstree.com systemd[1]: Started Network Time Service.
Jun 12 20:55:14 ninjaopstree.com ntpd[4635]: 0.0.0.0 c016 06 restart
Jun 12 20:55:14 ninjaopstree.com ntpd[4635]: 0.0.0.0 c012 02 freq_set kernel 0.000 PPM
Jun 12 20:55:14 ninjaopstree.com ntpd[4635]: 0.0.0.0 c011 01 freq_not_set
Jun 12 20:55:21 ninjaopstree.com ntpd[4635]: 0.0.0.0 c61c 0c clock_step +1.576376 s
Jun 12 20:55:22 ninjaopstree.com ntpd[4635]: 0.0.0.0 c614 04 freq_mode
Jun 12 20:55:23 ninjaopstree.com ntpd[4635]: 0.0.0.0 c618 08 no_sys_peer


We need to execute the ntpq -p command to verify NTP peers synchronization status.

[root@ninjaopstree ~]# ntpq -p
     remote           refid      st t when poll reach   delay   offset  jitter
==============================================================================
*13.67.76.75     218.73.139.35    2 u   24   64    1   88.840   45.353  26.839
 178.128.28.21   17.253.82.253    2 u   23   64    1   76.556   48.217  21.241
 time2.maxonline .INIT.          16 u    -   64    0    0.000    0.000   0.000
 139.59.219.101  118.189.138.5    2 u   31   64    1  153.068   12.514   0.000
 
 
 
 To find out the error we can create the dedicated log file.
 
 [root@ninjaopstree ~]# vim /etc/ntp.conf

Go to bottom of /etc/ntp.conf file and add the one configuration line
 
 #For NTP log
logfile /var/log/ntp.log




################################################################################################


Install java version 8 with home directory set as an environment variable

Download the Java 8 and extract in /opt directory

[root@ninjaopstree opt]# ls -ltr
total 194320
drwxr-xr-x. 8   10  143       255 Mar 29  2018 jdk1.8.0_171
drwxr-xr-x. 2 root root         6 Oct 31  2018 rh
drwxr-xr-x. 3 root root        19 Feb 18 18:34 remi
drwxr-xr-x. 2 3434 3434        56 Jun  4 22:21 node_exporter-0.18.1.linux-amd64
-rw-r--r--. 1 root root   8083296 Jun  4 22:21 node_exporter-0.18.1.linux-amd64.tar.gz
-rw-r--r--. 1 root root       225 Jun  9 15:36 index.html
-rw-r--r--. 1 root root 190890122 Jun 12 23:53 jdk-8u171-linux-x64.tar.gz



[root@ninjaopstree opt]# cd jdk1.8.0_171/


[root@ninjaopstree jdk1.8.0_171]# alternatives --install /usr/bin/java java /opt/jdk1.8.0_171/bin/java 2
[root@ninjaopstree jdk1.8.0_171]# alternatives --config java

There are 3 programs which provide 'java'.

  Selection    Command
-----------------------------------------------
 + 1           /opt/bin/java
*  2           java-1.8.0-openjdk.x86_64 (/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.212.b04-0.el7_6.x86_64/jre/bin/java)
   3           /opt/jdk1.8.0_171/bin/java

Enter to keep the current selection[+], or type selection number: 3\

There are 3 programs which provide 'java'.

  Selection    Command
-----------------------------------------------
 + 1           /opt/bin/java
*  2           java-1.8.0-openjdk.x86_64 (/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.212.b04-0.el7_6.x86_64/jre/bin/java)
   3           /opt/jdk1.8.0_171/bin/java

Enter to keep the current selection[+], or type selection number: 3


[root@ninjaopstree jdk1.8.0_171]# alternatives --install /usr/bin/jar jar /opt/jdk1.8.0_171/bin/jar 2
[root@ninjaopstree jdk1.8.0_171]# alternatives --install /usr/bin/javac javac /opt/jdk1.8.0_171/bin/javac 2
[root@ninjaopstree jdk1.8.0_171]# alternatives --set jar /opt/jdk1.8.0_171/bin/jar
[root@ninjaopstree jdk1.8.0_171]# alternatives --set javac /opt/jdk1.8.0_171/bin/javac
[root@ninjaopstree jdk1.8.0_171]# java -version
java version "1.8.0_171"
Java(TM) SE Runtime Environment (build 1.8.0_171-b11)
Java HotSpot(TM) 64-Bit Server VM (build 25.171-b11, mixed mode)
[root@ninjaopstree jdk1.8.0_171]#

Set the JAVA_HOME 

[root@ninjaopstree jdk1.8.0_171]# export JAVA_HOME=/opt/jdk1.8.0_171
[root@ninjaopstree jdk1.8.0_171]# export JRE_HOME=/opt/jdk1.8.0_171/jre
[root@ninjaopstree jdk1.8.0_171]# export PATH=$PATH:/opt/jdk1.8.0_171/bin:/opt/jdk1.8.0_171/jre/bin


[root@ninjaopstree jdk1.8.0_171]# echo $JAVA_HOME
/opt/jdk1.8.0_171
[root@ninjaopstree jdk1.8.0_171]#


#########################################################################################################


Install Tomcat version 8 (a brief explaination about the it's directories in doc)


[root@ninjaopstree opt]# wget https://www-us.apache.org/dist/tomcat/tomcat-8/v8.5.42/bin/apache-tomcat-8.5.42.tar.gz

[root@ninjaopstree opt]# tar -xzvf apache-tomcat-8.5.42.tar.gz
[root@ninjaopstree opt]# cd apache-tomcat-8.5.42
[root@ninjaopstree apache-tomcat-8.5.42]# cd bin/


[root@ninjaopstree bin]# ./startup.sh
Using CATALINA_BASE:   /opt/apache-tomcat-8.5.42
Using CATALINA_HOME:   /opt/apache-tomcat-8.5.42
Using CATALINA_TMPDIR: /opt/apache-tomcat-8.5.42/temp
Using JRE_HOME:        /opt/jdk1.8.0_171/jre
Using CLASSPATH:       /opt/apache-tomcat-8.5.42/bin/bootstrap.jar:/opt/apache-tomcat-8.5.42/bin/tomcat-juli.jar
Tomcat started.



#############################################################
Build-essentials is a reference for all packages which are considered essential for building Debian packages.
 So, if we want to be able to build Debian package,we need to install the build-essential package

To install the package, we just need to execute below command:

 sudo apt-get install build-essential

Note: We you can also issue the same command to install the build-essentials on Ubuntu

There is no “build-essentials” package in CentOS, RHEL. We can install the an equivalent one that need for building the software

yum install gcc gcc-c++ kernel-devel make

If we need a full build suite for CentOS, RHEL, we can install the Development Tools as below:
yum groupinstall "Development Tools"



###################################################################




NFS configuretion 

Step 1     Installed the nfs packages on the server side like nfs-utils nfs-utils-lib



[root@ninjaopstree ~]# yum install nfs-utils nfs-utils-lib

[root@ninjaopstree ~]# systemctl enable rpcbind

[root@ninjaopstree ~]# systemctl start rpcbind

[root@ninjaopstree ~]# systemctl enable nfs-server
Created symlink from /etc/systemd/system/multi-user.target.wants/nfs-server.service to /usr/lib/systemd/system/nfs-server.service.

[root@ninjaopstree ~]# systemctl enable nfs-lock

[root@ninjaopstree ~]# systemctl enable nfs-idmap

[root@ninjaopstree ~]# systemctl start nfs-server

[root@ninjaopstree ~]# systemctl start nfs-lock

[root@ninjaopstree ~]# systemctl start nfs-idmap


Now, let us create some shared directories in server.


[root@ninjaopstree ~]# systemctl start nfs-server
[root@ninjaopstree ~]# systemctl start nfs-lock
[root@ninjaopstree ~]# systemctl start nfs-idmap
[root@ninjaopstree ~]# mkdir /ninja
[root@ninjaopstree ~]# cd /ninja/
[root@ninjaopstree ninja]# touch ninja
[root@ninjaopstree ninja]#


[root@ninjaopstree /]# vi /etc/exports


/ninja     172.30.24.0/24(rw,sync,no_root_squash,no_all_squash)


where,

/ninja – shared directory
172.30.24.0/24 – IP address range of clients
rw – Writable permission to shared folder
sync – Synchronize shared directory
no_root_squash – Enable root privilege
no_all_squash - Enable user’s authority



[root@ninjaopstree /]# systemctl restart nfs-server
[root@ninjaopstree /]# systemctl status nfs-server
● nfs-server.service - NFS server and services
   Loaded: loaded (/usr/lib/systemd/system/nfs-server.service; enabled; vendor preset: disabled)
   Active: active (exited) since Thu 2019-06-13 17:01:52 IST; 5s ago
  Process: 4304 ExecStopPost=/usr/sbin/exportfs -f (code=exited, status=0/SUCCESS)
  Process: 4301 ExecStopPost=/usr/sbin/exportfs -au (code=exited, status=0/SUCCESS)
  Process: 4300 ExecStop=/usr/sbin/rpc.nfsd 0 (code=exited, status=0/SUCCESS)
  Process: 4331 ExecStartPost=/bin/sh -c if systemctl -q is-active gssproxy; then systemctl restart gssproxy ; fi (code=exited, status=0/SUCCESS)
  Process: 4314 ExecStart=/usr/sbin/rpc.nfsd $RPCNFSDARGS (code=exited, status=0/SUCCESS)
  Process: 4313 ExecStartPre=/usr/sbin/exportfs -r (code=exited, status=0/SUCCESS)
 Main PID: 4314 (code=exited, status=0/SUCCESS)
   CGroup: /system.slice/nfs-server.service

Jun 13 17:01:52 ninjaopstree.com systemd[1]: Starting NFS server and services...
Jun 13 17:01:52 ninjaopstree.com systemd[1]: Started NFS server and services.
[root@ninjaopstree /]#


Now i have mapped the /ninja share drive on client machine

[root@ninja ~]# vim /etc/yum.repos.d/epel.repo

[root@ninja ~]# /etc/init.d/rpcbind start
Starting rpcbind:                                          [  OK  ]

[root@ninja ~]# /etc/init.d/nfs start
Starting NFS services:                                     [  OK  ]
Starting NFS mountd:                                       [  OK  ]
Starting NFS daemon:                                       [  OK  ]
Starting RPC idmapd:                                       [  OK  ]


All the required service runing fine now i have to check the mount point of the server

[root@ninja ~]# showmount -e 172.30.24.107
Export list for 172.30.24.107:
/ninja 172.30.24.0/24


Now mount the shared drive /mnt

[root@ninja ~]# mount -t nfs 172.30.24.107:/ninja /mnt

[root@ninja ~]# mount -a

finally to checked the nfs drive is mapped on not.

[root@ninja ~]# df -h
Filesystem                    Size  Used Avail Use% Mounted on
/dev/mapper/VolGroup-lv_root   21G  2.2G   17G  12% /
tmpfs                         939M     0  939M   0% /dev/shm
/dev/sda1                     485M   33M  427M   8% /boot
172.30.24.107:/ninja           22G  3.8G   18G  18% /mnt


Verifying NFS Shares On Clients


[root@ninja mnt]# mount
/dev/mapper/VolGroup-lv_root on / type ext4 (rw)
proc on /proc type proc (rw)
sysfs on /sys type sysfs (rw)
devpts on /dev/pts type devpts (rw,gid=5,mode=620)
tmpfs on /dev/shm type tmpfs (rw)
/dev/sda1 on /boot type ext4 (rw)
none on /proc/sys/fs/binfmt_misc type binfmt_misc (rw)
nfsd on /proc/fs/nfsd type nfsd (rw)
sunrpc on /var/lib/nfs/rpc_pipefs type rpc_pipefs (rw)
172.30.24.107:/ninja on /mnt type nfs (rw,vers=4,addr=172.30.24.107,clientaddr=172.30.24.121)


See the content !!

[root@ninja mnt]# ls -ltr
total 4
-rw-r--r-- 1 root root 29 Jun 13 17:08 devops

its mapped the and working fine.

Thanks!



Apart from that i have installed the tomcat run as a service in centos below are the steps.

Fristly create a user name with tomcat with home directly and nologin shell



  [root@ninjaopstree opt]# useradd -M -s /bin/nologin -g tomcat -d /opt/tomcat tomcat
  
 [root@ninjaopstree opt]# wget https://www-us.apache.org/dist/tomcat/tomcat-8/v8.5.42/bin/apache-tomcat-8.5.42.tar.gz
 
 [root@ninjaopstree /]# mkdir /opt/tomcat
 
 [root@ninjaopstree /]# tar xvf apache-tomcat-8*tar.gz -C /opt/tomcat --strip-components=1
 
 [root@ninjaopstree /]# cd /opt/tomcat


  [root@ninjaopstree opt]#  cat /etc/group

 [root@ninjaopstree opt]#  chgrp -R tomcat /opt/apache-tomcat-8.5.42
 [root@ninjaopstree opt]#  cd /opt/apache-tomcat-8.5.42

 [root@ninjaopstree opt]# chmod -R g+r conf
 [root@ninjaopstree opt]# chmod g+x conf

 [root@ninjaopstree opt]# chown -R tomcat webapps/ work/ temp/ logs/

  Install Systemd Unit File
  
  [root@ninjaopstree /]# vim /etc/systemd/system/tomcat.service
  
  # Systemd unit file for tomcat
[Unit]
Description=Apache Tomcat Web Application Container
After=syslog.target network.target

[Service]
Type=forking

Environment=JAVA_HOME=/opt/jdk1.8.0_171   ####################################  Changed the JAVA_HOME Veriable path

Environment=CATALINA_PID=/opt/tomcat/temp/tomcat.pid
Environment=CATALINA_HOME=/opt/tomcat
Environment=CATALINA_BASE=/opt/tomcat
Environment='CATALINA_OPTS=-Xms512M -Xmx1024M -server -XX:+UseParallelGC'
Environment='JAVA_OPTS=-Djava.awt.headless=true -Djava.security.egd=file:/dev/./urandom'

ExecStart=/opt/tomcat/bin/startup.sh
ExecStop=/bin/kill -15 $MAINPID

User=tomcat
Group=tomcat
UMask=0007
RestartSec=10
Restart=always

[Install]
WantedBy=multi-user.target

[root@ninjaopstree /]# systemctl daemon-reload

[root@ninjaopstree /]# systemctl start tomcat


I m also read the follwing files:

[root@ninjaopstree conf]# ls -ltr
total 224
-rw-r-----. 1 root tomcat 171082 Jun  5 02:01 web.xml
-rw-r-----. 1 root tomcat   2633 Jun  5 02:01 tomcat-users.xsd
-rw-r-----. 1 root tomcat   7511 Jun  5 02:01 server.xml
-rw-r-----. 1 root tomcat   3916 Jun  5 02:01 logging.properties
-rw-r-----. 1 root tomcat   2313 Jun  5 02:01 jaspic-providers.xsd
-rw-r-----. 1 root tomcat   1149 Jun  5 02:01 jaspic-providers.xml
-rw-r-----. 1 root tomcat   7661 Jun  5 02:01 catalina.properties
-rw-r-----. 1 root tomcat  13548 Jun  5 02:01 catalina.policy
-rw-r-----. 1 root tomcat   1339 Jun 13 12:39 context.xml
-rw-r-----. 1 root tomcat   1149 Jun 13 12:44 tomcat-users.xml

Thanks !!!



######################################################################

Install git (a brief explaination about - what it is and why do we need it in doc)


[root@ninjaopstree ~]# yum install git


[root@ninjaopstree ~]# git --version
git version 1.8.3.1


Git is a free and open source, fast and distributed version control system (VCS), 
which by design is based on speed, efficient performance and data integrity to support small-scale to extensive software development projects.


Easy to learn
It is fast and most of its operations are carried out locally, 
in addition, this offers it a tremendous speed on centralized systems that need to communicate with remote servers.

Highly efficient
Supports data integrity checks
Enables cheap local branching
Offers a convenient staging area
It also maintains multiple work-flows together with many others

Git is a software repository that allows you to keep a track of your software changes, revert to previous version and 
create another versions of files and directories.

Git is written in C, with a mix of Perl and a variety of shell scripts.
 it’s primarily intended to run on the Linux kernel and has a number of remarkable features as listed below:










