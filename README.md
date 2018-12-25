# boss-ansible-role-nfs
Configure nfs

# FOLLOW THIS GUIDE!!!!!!!!!!!!!!!!!!

* https://websiteforstudents.com/setup-nfs-mounts-on-ubuntu-16-04-lts-servers-for-client-computers-to-access/
https://websiteforstudents.com/setup-nfs-mounts-on-ubuntu-16-04-lts-servers-for-client-computers-to-access/
* https://websiteforstudents.com/setup-nfs-mounts-on-ubuntu-16-04-lts-servers-for-client-computers-to-access/
https://websiteforstudents.com/setup-nfs-mounts-on-ubuntu-16-04-lts-servers-for-client-computers-to-access/
* https://websiteforstudents.com/setup-nfs-mounts-on-ubuntu-16-04-lts-servers-for-client-computers-to-access/
https://websiteforstudents.com/setup-nfs-mounts-on-ubuntu-16-04-lts-servers-for-client-computers-to-access/
* https://websiteforstudents.com/setup-nfs-mounts-on-ubuntu-16-04-lts-servers-for-client-computers-to-access/
https://websiteforstudents.com/setup-nfs-mounts-on-ubuntu-16-04-lts-servers-for-client-computers-to-access/


# perf stuff to try

```
 - path: /etc/modules-load.d/nf.conf
    content: |
      nf_conntrack
  - path: /etc/sysctl.d/ker_optimizations.conf
    content: |
      net.core.somaxconn=65535
      net.ipv4.ip_local_port_range=10240  63999
      net.ipv4.tcp_tw_recycle=0
      net.ipv4.tcp_tw_reuse=1
      net.ipv4.tcp_max_syn_backlog=65535
      net.ipv4.tcp_max_tw_buckets=1440000
      net.ipv4.tcp_window_scaling=1
      net.ipv4.tcp_congestion_control=htcp
      net.core.netdev_max_backlog=250000
      net.ipv4.tcp_mtu_probing=1
      net.ipv4.tcp_slow_start_after_idle=0
      fs.file-max=6511048
      net.netfilter.nf_conntrack_max=262144
      net.ipv4.tcp_max_syn_backlog=4096
and this change

+    LimitNPROC=1048576
 +    LimitCORE=infinity
```

https://security.stackexchange.com/questions/43205/nf-conntrack-table-full-dropping-packet
```
The message means your connection tracking table is full. There are no security implications other than DoS. You can partially mitigate this by increasing the maximum number of connections being tracked, reducing the tracking timeouts or by disabling connection tracking altogether, which is doable on server, but not on a NAT router, because the latter will cease to function.

sysctl -w net.ipv4.netfilter.ip_conntrack_tcp_timeout_established=54000
sysctl -w net.netfilter.nf_conntrack_generic_timeout=120
sysctl -w net.ipv4.netfilter.ip_conntrack_max=<more than currently set>
```


# Perf

http://www.brendangregg.com/ebpf#bcc


# Evolution of NFS Versions

NFS version 2 only allowed access to the first 2 GB of a file. The client made the decision about whether the user could access the file or not. This version supported UDP only, and it limited write operations to 8 kbytes. Ugh. It was a nice start, though.

NFS version 3 supported much larger files. The client could ask the server to make the access decision — although it didn't have to and so we still had to fully trust all client hosts. NFSv3 over UDP could use up to 56 kbyte transactions. There was now support for NFS over TCP, although client implementations varied quite a bit as to details with associated compatibility issues.

NFS version 4 added statefulness, so an NFSv4 client could directly inform the server about its locking, reading, writing the file, and so on. All the NFS application protocols — mount, stat, ACL checking, and the NFS file service itself, run over a single network protocol always running on a predictable TCP port 2049. There is no longer any need for an RPC port mapper to start first, or for separate mount and file locking daemons to run independently. Access control is now done by user name, not possibly inconsistent UID and GID as before, and security could be further enhanced with the use of Kerberos.

NFS version 4.1 adds Parallel NFS (or pNFS), which can stripe data across multiple NFS servers. You can do block-level access and use NFS v4.1 much like Fibre Channel and iSCSI, and object access is meant to be analogous to AWS S3. NFS v4.1 adds some performance enhancements Perhaps more importantly to many current users, NFS v4.1 on Linux better inter-operates with non-Linux NFS servers and clients. Specifically, mounting NFS from Windows servers using Kerberos V5. For all the details see RFC 5661.

# Perf NFS tools

SOURCE: https://cromwell-intl.com/open-source/performance-tuning/nfs.html

NFS Performance Goals
Rotating SATA and SAS disks can provide I/O of about 1 Gbps. Solid-state disks can fill the 6 Gbps bandwidth of a SATA 3 controller. NFS should be about this fast, filling a 1 Gbps LAN. NFS SAN appliances can fill a 10 Gbps LAN. Random I/O performance should be good, about 100 transactions per second per spindle.

# OPTIONS:
* -d     the directory to use for the tests
* -s     the  size  of the file(s) for IO performance measures in megabytes. If the size is greater than 1G then multiple files will be used to store the data, and each file will be up to 1G in size.  This parameter may include the chunk size seperated from the size by a colon.  The chunk-size is measured in bytes and must be a power of two from 256 to 1048576, the default is 8192. NB You can specify the size in giga-bytes or the  chunk-size in kilo-bytes if you add g or k to the end of the number respectively. If the specified size is 0 then this test will be skipped.
* -r     RAM  size  in megabytes. If you specify this the other parameters will be checked to ensure they make sense for a machine of that much RAM. You should not need to do this in general use as it should be able to discover the RAM size. NB If you specify a size of 0 then all checks will be disabled...
* -u     user-id to use.  When running as root specify the UID to use for the tests.  It is not recommended to use root (since the occasion when a Bonnie++ bug wiped out someone's system), so if you really want to run  as  root then  use -u root.  Also if you want to specify the group to run as then use the user:group format.  If you specify a user by name but no group then the primary group of that user will be chosen.  If you specify a user by number and no group then the group will be nogroup.
* -g     group-id to use.  Same as using :group for the -u parameter, just a different way to specify it for compatibility with other programs.


```
bonnie++ -d /mnt/publicdata/ -s 2048 -r 1024 -u 65534 -g 65534
```

### Example output

```
root@nfs-worker1:~# bonnie++ -d /mnt/publicdata/ -s 2048 -r 1024 -u 65534 -g 65534
Using uid:65534, gid:65534.
Writing a byte at a time...done
Writing intelligently...done
Rewriting...done
Reading a byte at a time...done
Reading intelligently...done
start 'em...done...done...done...done...done...
Create files in sequential order...done.
Stat files in sequential order...done.
Delete files in sequential order...done.
Create files in random order...done.
Stat files in random order...done.
Delete files in random order...done.
Version  1.97       ------Sequential Output------ --Sequential Input- --Random-
Concurrency   1     -Per Chr- --Block-- -Rewrite- -Per Chr- --Block-- --Seeks--
Machine        Size K/sec %CP K/sec %CP K/sec %CP K/sec %CP K/sec %CP  /sec %CP
nfs-worker1      2G  1113  97 78993   7 75546  11  2102  92 271440  13  6863  53
Latency             21103us     801ms    1565ms   19930us   21978us    9956us
Version  1.97       ------Sequential Create------ --------Random Create--------
nfs-worker1         -Create-- --Read--- -Delete-- -Create-- --Read--- -Delete--
              files  /sec %CP  /sec %CP  /sec %CP  /sec %CP  /sec %CP  /sec %CP
                 16   407   7 +++++ +++  1328  12   526   6 +++++ +++  1704  13
Latency             65891us   18799us   81134us   31558us    3596us   10771us
1.97,1.97,nfs-worker1,1,1545782353,2G,,1113,97,78993,7,75546,11,2102,92,271440,13,6863,53,16,,,,,407,7,+++++,+++,1328,12,526,6,+++++,+++,1704,13,21103us,801ms,1565ms,19930us,21978us,9956us,65891us,18799us,81134us,31558us,3596us,10771us
root@nfs-worker1:~#
```


```
# iozone -aRcU /mnt/nfs/ -f /mnt/nfs/testfile > logfile
```

# compare mount configs to blog post

```
/mnt/publicdata from 192.168.50.101:/mnt/publicdata
 Flags:	rw,noatime,vers=4.0,rsize=262144,wsize=262144,namlen=255,acregmin=1800,acregmax=1800,acdirmin=1800,acdirmax=1800,hard,proto=tcp,port=0,timeo=600,retrans=2,sec=sys,clientaddr=192.168.50.102,local_lock=none,addr=192.168.50.101
```


https://cromwell-intl.com/open-source/performance-tuning/disks.html
https://linuxconfig.org/how-to-automatically-execute-shell-script-at-startup-boot-on-systemd-linux
