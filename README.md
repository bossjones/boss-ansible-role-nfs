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
