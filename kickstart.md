# 1.部署前准备

```shell
yum install -y createrepo   mkisofs pykickstart
ksvalidator ks7_minimal.cfg
```



## CentOS6本地安装配置文件

```shell
# Kickstart file automatically generated by anaconda.

#version=DEVEL
install
cmdline
cdrom
#url --url=https://mirrors.aliyun.com/centos/6/os/x86_64/
text
lang en_US.UTF-8
keyboard us
network --onboot yes --device eth0 --bootproto dhcp --noipv6
network --onboot yes --device eth1 --bootproto dhcp --noipv6
rootpw  --iscrypted $6$PpSMzO3EiKR4hp9o$1D.tioP5/IdDz5xd7tkoMuh0NdRlLGLjOfrPMRJ9BUryZ8XHcoNW5ezmRpoob9gzPo7REOJXvgweN2SxTBU3P0
firewall --disable
authconfig --enableshadow --passalgo=sha512
selinux --disable
reboot
skipx
timezone Asia/Shanghai
bootloader --location=mbr --driveorder=sda --append="crashkernel=auto rhgb quiet"
# The following is the partition information you requested
# Note that any partitions you deleted are not expressed
# here so unless you clear all partitions first, this is
# not guaranteed to work
clearpart --all --initlabel
zerombr
part /boot --fstype=ext4 --size=200
part pv.008002 --grow --maxsize=30720 --size=200

volgroup sysvg --pesize=4096 pv.008002
logvol / --fstype=ext4 --name=lv_root --vgname=sysvg --size=28668
logvol swap --name=lv_swap --vgname=sysvg --size=2048

%packages --nobase
@core
%post
cd /etc/selinux/ && tar -czf selinux_bak.tar.gz ./*
cd /etc/yum.repos.d/ && tar -czf yum_repo_bak.tar.gz ./* ; rm -f *.repo
cd /etc/sysconfig/network-scripts && tar -czf network.tar.gz ./*
cd /etc/ssh/ && tar -czf ssh_tar.gz ./*
setenforce 0
chkconfig postfix off
chkconfig ip6tables off
chkconfig iptables off 
service ip6tables stop
service iptables stop 
service postfix stop         
sed -i -e 's/#UseDNS yes/UseDNS no/g' -e 's/GSSAPIAuthentication yes/GSSAPIAuthentication no/g' /etc/ssh/sshd_config
sed -i 's/SELINUX=enforcing/SELINUX=disabled/' /etc/selinux/config
service sshd restart
cat >> /root/repo_create.sh << EOF
curl -o /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-6.repo
sed -i -e '/mirrors.cloud.aliyuncs.com/d' -e '/mirrors.aliyuncs.com/d' /etc/yum.repos.d/CentOS-Base.repo
curl -o  /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-6.repo
sed -i 's|^#baseurl=https://download.fedoraproject.org/pub|baseurl=https://mirrors.aliyun.com|' /etc/yum.repos.d/epel*
sed -i 's|^metalink|#metalink|' /etc/yum.repos.d/epel*
EOF

cat >> /root/yum_install.sh << EOF
yum clean all &>/dev/null
yum makecache &>/dev/null
yum install -y lrzsz telnet wget chrony tree lsof tcpdump dos2unix net-tools vim &>/dev/null
chkconfig --add chronyd
chkconfig chronyd on
service chronyd start
EOF
chmod u+x /root/repo_create.sh
chmod u+x /root/yum_install.sh
%end
```

## CentOS7本地安装配置文件  

```shell
#version=DEVEL
# System authorization information
auth --enableshadow --passalgo=sha512
# Use CDROM installation media
cdrom
#url --url=https://mirrors.aliyun.com/centos/7/os/x86_64/
# Use text mode install
text
# Run the Setup Agent on first boot
firstboot --enable
ignoredisk --only-use=sda
# Keyboard layouts
keyboard --vckeymap=us --xlayouts='us'
# System language
lang en_US.UTF-8

# Reboot after installation
reboot

# Network information
network  --bootproto=dhcp --device=ens32 --ipv6=auto --activate
network  --bootproto=dhcp --device=ens33 --ipv6=auto --activate
network  --hostname=Centos7

# Root password
rootpw --iscrypted $6$usDw9KY8T2Zkg7/S$hsDF.PkrCCsvhMYcSgaJ.bZ2PJ2uESaG2pN3gl8QmlZKjq44UJO4RI3t/fINX0TABC8EwFsa0tAW.0/aVzB0T.
# System services
# SELinux configuration
selinux --disabled
# System timezone
timezone Asia/Shanghai --isUtc --nontp
# System bootloader configuration
bootloader --append=" crashkernel=auto" --location=mbr --boot-drive=sda
# Partition clearing information
# clearpart --none --initlabel
clearpart --all --initlabel
# Disk partitioning information
part /boot --fstype=ext4 --size=200
part pv.008002 --grow --maxsize=30720 --size=200

volgroup sysvg --pesize=4096 pv.008002
logvol / --fstype=ext4 --name=lv_root --vgname=sysvg --size=28668
logvol swap --name=lv_swap --vgname=sysvg --size=2048



%packages
@^minimal
@core
kexec-tools

%end

%post
cd /etc/selinux/ && tar -czf selinux_bak.tar.gz ./*
cd /etc/yum.repos.d/ && tar -czf yum_repo_bak.tar.gz ./* ; rm -f *.repo
cd /etc/sysconfig/network-scripts && tar -czf network.tar.gz ./*
cd /etc/ssh/ && tar -czf ssh_tar.gz ./*
setenforce 0
sed -i -e 's/#UseDNS yes/UseDNS no/g' -e 's/GSSAPIAuthentication yes/GSSAPIAuthentication no/g' /etc/ssh/sshd_config
sed -i 's/SELINUX=enforcing/SELINUX=disabled/' /etc/selinux/config
systemctl restart sshd
systemctl disable --now  postfix
systemctl disable --now firewalld

cat >> /root/repo_create.sh << EOF
curl -o /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-7.repo
curl -o /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-7.repo
sed -i -e '/mirrors.cloud.aliyuncs.com/d' -e '/mirrors.aliyuncs.com/d' /etc/yum.repos.d/CentOS-Base.repo
sed -i 's|^#baseurl=https://download.fedoraproject.org/pub|baseurl=https://mirrors.aliyun.com|' /etc/yum.repos.d/epel*
sed -i 's|^metalink|#metalink|' /etc/yum.repos.d/epel*
EOF

cat >> /root/yum_install.sh << EOF
yum clean all &>/dev/null
yum makecache &>/dev/null
yum install -y lrzsz telnet wget chrony tree lsof tcpdump dos2unix net-tools vim &>/dev/null
systemctl enable --now chronyd
EOF

chmod u+x /root/repo_create.sh
chmod u+x /root/yum_install.sh

%end

%addon com_redhat_kdump --enable --reserve-mb='auto'

%end

%anaconda
pwpolicy root --minlen=6 --minquality=1 --notstrict --nochanges --notempty
pwpolicy user --minlen=6 --minquality=1 --notstrict --nochanges --emptyok
pwpolicy luks --minlen=6 --minquality=1 --notstrict --nochanges --notempty
%end
```

# CentOS6部署kickstart文件到iso镜像

```shell
1.挂载光盘镜像
[root@localhost /]# mount /dev/cdrom /mnt
mount: block device /dev/sr0 is write-protected, mounting read-only
2.复制已挂载的镜像文件到/myiso下
[root@localhost /]# 
[root@localhost /]# cp -aR /mnt/* myiso/
[root@localhost /]# 
[root@localhost /]# ls /myiso/
CentOS_BuildTag  EULA  images    Packages                  repodata              RPM-GPG-KEY-CentOS-Debug-6     RPM-GPG-KEY-CentOS-Testing-6
EFI              GPL   isolinux  RELEASE-NOTES-en-US.html  RPM-GPG-KEY-CentOS-6  RPM-GPG-KEY-CentOS-Security-6  TRANS.TBL
[root@localhost /]# 
3.准备kickstart配置文件
[root@localhost /]# mkdir /myiso/ks.d
[root@localhost /]# cp /root/anaconda-ks.cfg /myiso/ks.d
[root@localhost /]# vi /myiso/ks.d/anaconda-ks.cfg 
[root@localhost /]# mv /myiso/ks.d/anaconda-ks.cfg /myiso/ks.d/ks6_minimal.cfg
[root@localhost /]# cat /myiso/ks.d/ks6_minimal.cfg
# Kickstart file automatically generated by anaconda.

#version=DEVEL
install
cmdline
cdrom
text
lang en_US.UTF-8
keyboard us
network --onboot yes --device eth0 --bootproto dhcp --noipv6
network --onboot yes --device eth1 --bootproto dhcp --noipv6
rootpw  --iscrypted $6$PpSMzO3EiKR4hp9o$1D.tioP5/IdDz5xd7tkoMuh0NdRlLGLjOfrPMRJ9BUryZ8XHcoNW5ezmRpoob9gzPo7REOJXvgweN2SxTBU3P0
firewall --disable
authconfig --enableshadow --passalgo=sha512
selinux --disable
reboot
skipx
timezone Asia/Shanghai
bootloader --location=mbr --driveorder=sda --append="crashkernel=auto rhgb quiet"
# The following is the partition information you requested
# Note that any partitions you deleted are not expressed
# here so unless you clear all partitions first, this is
# not guaranteed to work
clearpart --all --initlabel
zerombr
part /boot --fstype=ext4 --size=200
part pv.008002 --grow --maxsize=30720 --size=200

volgroup sysvg --pesize=4096 pv.008002
logvol / --fstype=ext4 --name=lv_root --vgname=sysvg --size=28668
logvol swap --name=lv_swap --vgname=sysvg --size=2048



%packages --nobase
@core
%post
/bin/sed -ri 'sSexec /sbin/shutdown -r now.*S#&Sg' /etc/init/control-alt-delete.conf 
cd /etc/selinux/ && tar -czf selinux_bak.tar.gz ./*
cd /etc/yum.repos.d/ && tar -czf yum_repo_bak.tar.gz ./* && rm -f *.repo
cd /etc/sysconfig/network-scripts && tar -czf network.tar.gz ./*
cd /etc/ssh/ && tar -czf ssh_tar.gz ./*
setenforce 0
chkconfig postfix off
chkconfig ip6tables off
chkconfig iptables off
service ip6tables stop
service iptables stop
service postfix stop
sed -i -e 's/#UseDNS yes/UseDNS no/g' -e 's/GSSAPIAuthentication yes/GSSAPIAuthentication no/g' /etc/ssh/sshd_config
sed -i 's/SELINUX=enforcing/SELINUX=disabled/' /etc/selinux/config
service sshd restart
cat >> /root/repo_create.sh << EOF
curl -o /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-6.repo
sed -i -e '/mirrors.cloud.aliyuncs.com/d' -e '/mirrors.aliyuncs.com/d' /etc/yum.repos.d/CentOS-Base.repo
curl -o  /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-6.repo
sed -i 's|^#baseurl=https://download.fedoraproject.org/pub|baseurl=https://mirrors.aliyun.com|' /etc/yum.repos.d/epel*
sed -i 's|^metalink|#metalink|' /etc/yum.repos.d/epel*
EOF

cat >> /root/yum_install.sh << EOF
yum clean all &>/dev/null
yum makecache &>/dev/null
yum install -y lrzsz telnet wget  tree lsof tcpdump dos2unix net-tools vim &>/dev/null
EOF

chmod u+x /root/repo_create.sh
chmod u+x /root/yum_install.sh

%end
[root@localhost /]#
5.修改isolinux/isolinux.cfg配置
[root@localhost /]# cd /myiso/
[root@localhost myiso]# 
[root@localhost myiso]# ls
CentOS_BuildTag  EULA  images    ks.d      RELEASE-NOTES-en-US.html  RPM-GPG-KEY-CentOS-6        RPM-GPG-KEY-CentOS-Security-6  TRANS.TBL
EFI              GPL   isolinux  Packages  repodata                  RPM-GPG-KEY-CentOS-Debug-6  RPM-GPG-KEY-CentOS-Testing-6
[root@localhost myiso]# cd isolinux/
[root@localhost isolinux]# ls
boot.cat  boot.msg  grub.conf  initrd.img  isolinux.bin  isolinux.cfg  memtest  splash.jpg  TRANS.TBL  vesamenu.c32  vmlinuz
[root@localhost isolinux]# vi isolinux.cfg 
[root@localhost isolinux]# cat isolinux.cfg
default vesamenu.c32
#prompt 1
timeout 100

display boot.msg

menu background splash.jpg
menu title Welcome to CentOS 6.10!
menu color border 0 #ffffffff #00000000
menu color sel 7 #ffffffff #ff000000
menu color title 0 #ffffffff #00000000
menu color tabmsg 0 #ffffffff #00000000
menu color unsel 0 #ffffffff #00000000
menu color hotsel 0 #ff000000 #ffffffff
menu color hotkey 7 #ffffffff #ff000000
menu color scrollbar 0 #ffffffff #00000000

label linux
  menu label ^Install or upgrade an existing system
  menu default
  kernel vmlinuz
  append initrd=initrd.img
label minimal
  menu label ^install CentOS6 auto
  kernel vmlinuz
  append initrd=initrd.img ks=cdrom:/ks.d/ks6_minimal.cfg
label vesa
  menu label Install system with ^basic video driver
  kernel vmlinuz
  append initrd=initrd.img nomodeset
label rescue
  menu label ^Rescue installed system
  kernel vmlinuz
  append initrd=initrd.img rescue
label local
  menu label Boot from ^local drive
  localboot 0xffff
label memtest86
  menu label ^Memory test
  kernel memtest
  append -

[root@localhost isolinux]#
6.删除TRANS.TBL，重做repodata
[root@localhost myiso]# find ./ -name "TRANS.TBL"
./Packages/TRANS.TBL
./TRANS.TBL
./EFI/TRANS.TBL
./EFI/BOOT/TRANS.TBL
./images/TRANS.TBL
./images/pxeboot/TRANS.TBL
./isolinux/TRANS.TBL
./repodata/TRANS.TBL
[root@localhost myiso]# find ./ -name "TRANS.TBL" -exec rm -rf {} \;
[root@localhost myiso]#
[root@localhost myiso]# cp repodata/
34bae2d3c9c78e04ed2429923bc095005af1b166d1a354422c4c04274bae0f59-c6-minimal-x86_64.xml     cfc26bd7236d4f2701483b2aa11d1b476ecfd82006118194ab843b119e0d9837-filelists.sqlite.bz2
3a3fdf21c30413d0ba16d2a1df7749714c141d5a168bfc722e525abd25452c95-filelists.xml.gz          e5a44c63ccaa964a8172ec07ef37ccfa3cf37964c64149fbf7bd1be7c0b1ff53-other.xml.gz
3f1eaad4873142245acdc927bc6dd2c1a773c58197051487323a47a981718bcd-other.sqlite.bz2          fd5ad05899a142bdafb576d79dfa318f7ba0a6fb165fbc38a5a3d4df247ce531-primary.xml.gz
531b1f71184d2cf45277bb3c0a6f1e4ac4ad512368492b3b8cdea148904f4cb9-primary.sqlite.bz2        repomd.xml
ce2d698b9fb1413b668443e88835a0642cea8f387c7f25cc946f56dd93f109bb-c6-minimal-x86_64.xml.gz  repomd.xml.asc
[root@localhost myiso]# cp repodata/34bae2d3c9c78e04ed2429923bc095005af1b166d1a354422c4c04274bae0f59-c6-minimal-x86_64.xml /root
[root@localhost myiso]# rm -rf repodata/
[root@localhost myiso]# 
[root@localhost myiso]# createrepo -g /root/34bae2d3c9c78e04ed2429923bc095005af1b166d1a354422c4c04274bae0f59-c6-minimal-x86_64.xml /myiso/
Spawning worker 0 with 249 pkgs
Workers Finished
Gathering worker results

Saving Primary metadata
Saving file lists metadata
Saving other metadata
Generating sqlite DBs
Sqlite DBs complete
[root@localhost myiso]# ls repodata/
2223e566c4982ccafbd62d7505b3ea173cb4e860267dcddfe0ca4656289da92d-filelists.xml.gz       ce2d698b9fb1413b668443e88835a0642cea8f387c7f25cc946f56dd93f109bb-c6-minimal-x86_64.xml.gz
2e5c551ee51a637f180939ac84b2f4003738bae57e323950ce9f88c4a2b0ff4d-filelists.sqlite.bz2   d16cf58be4a3566a58f64b5962d4c3d72d6280f8998f6059535d51f4a5184ddb-other.sqlite.bz2
34bae2d3c9c78e04ed2429923bc095005af1b166d1a354422c4c04274bae0f59-c6-minimal-x86_64.xml  e97fc8efd35d5fa15a214bf6341a8f25653019173d62f9d7e0e51f57c7aef1cb-primary.xml.gz
9013c340fa58b060aef61fd91da82df66cf443badbdb0502c3356331d6153fe9-primary.sqlite.bz2     repomd.xml
a4a5d17dbae34d595e78c048b663ba4a531296dbb906756c2e83b0b71c1be2bc-other.xml.gz
[root@localhost myiso]#
7.iso镜像制作
[root@localhost myiso]# ls
CentOS_BuildTag  EULA  images    ks.d      RELEASE-NOTES-en-US.html  RPM-GPG-KEY-CentOS-6        RPM-GPG-KEY-CentOS-Security-6
EFI              GPL   isolinux  Packages  repodata                  RPM-GPG-KEY-CentOS-Debug-6  RPM-GPG-KEY-CentOS-Testing-6
[root@localhost myiso]# cd ..
[root@localhost /]# mkisofs -R -J -T -v --no-emul-boot --boot-load-size 4 --boot-info-table -V "CentOS-6.10-x86_64-minimal"  -b isolinux/isolinux.bin -c isolinux/boot.cat -o /root/CentOS-6.10_x86_64_auto_install.iso /myiso
I: -input-charset not specified, using utf-8 (detected in locale settings)
genisoimage 1.1.9 (Linux)
Scanning /myiso/
Scanning /myiso/Packages
Scanning /myiso/EFI
Scanning /myiso/EFI/BOOT
Scanning /myiso/ks.d
Scanning /myiso/images
Scanning /myiso/images/pxeboot
Scanning /myiso/isolinux
Excluded by match: /myiso/isolinux/boot.cat
Scanning /myiso/repodata
Using RPM_G000.;1 for  /RPM-GPG-KEY-CentOS-Debug-6 (RPM-GPG-KEY-CentOS-Security-6)
Using RPM_G001.;1 for  /RPM-GPG-KEY-CentOS-Security-6 (RPM-GPG-KEY-CentOS-Testing-6)
Using RPM_G002.;1 for  /RPM-GPG-KEY-CentOS-Testing-6 (RPM-GPG-KEY-CentOS-6)
Using DEVIC000.RPM;1 for  /myiso/Packages/device-mapper-multipath-libs-0.4.9-106.el6.x86_64.rpm (device-mapper-event-libs-1.02.117-12.el6_9.1.x86_64.rpm)
Using LIBSE000.RPM;1 for  /myiso/Packages/libselinux-2.0.94-7.el6.x86_64.rpm (libselinux-utils-2.0.94-7.el6.x86_64.rpm)
Using DEVIC001.RPM;1 for  /myiso/Packages/device-mapper-event-libs-1.02.117-12.el6_9.1.x86_64.rpm (device-mapper-event-1.02.117-12.el6_9.1.x86_64.rpm)
Using DEVIC002.RPM;1 for  /myiso/Packages/device-mapper-event-1.02.117-12.el6_9.1.x86_64.rpm (device-mapper-multipath-0.4.9-106.el6.x86_64.rpm)
Using KEYUT000.RPM;1 for  /myiso/Packages/keyutils-1.4-5.el6.x86_64.rpm (keyutils-libs-1.4-5.el6.x86_64.rpm)
Using FIPSC000.RPM;1 for  /myiso/Packages/fipscheck-1.2.0-7.el6.x86_64.rpm (fipscheck-lib-1.2.0-7.el6.x86_64.rpm)
Using NFS_U000.RPM;1 for  /myiso/Packages/nfs-utils-lib-1.1.5-13.el6.x86_64.rpm (nfs-utils-1.2.3-78.el6.x86_64.rpm)
Using IPTAB000.RPM;1 for  /myiso/Packages/iptables-1.4.7-19.el6.x86_64.rpm (iptables-ipv6-1.4.7-19.el6.x86_64.rpm)
Using P11_K000.RPM;1 for  /myiso/Packages/p11-kit-0.18.5-2.el6_5.2.x86_64.rpm (p11-kit-trust-0.18.5-2.el6_5.2.x86_64.rpm)
Using NCURS000.RPM;1 for  /myiso/Packages/ncurses-base-5.7-4.20090207.el6.x86_64.rpm (ncurses-libs-5.7-4.20090207.el6.x86_64.rpm)
Using PLYMO000.RPM;1 for  /myiso/Packages/plymouth-0.8.3-29.el6.centos.x86_64.rpm (plymouth-scripts-0.8.3-29.el6.centos.x86_64.rpm)
Using COREU000.RPM;1 for  /myiso/Packages/coreutils-libs-8.4-47.el6.x86_64.rpm (coreutils-8.4-47.el6.x86_64.rpm)
Using CYRUS000.RPM;1 for  /myiso/Packages/cyrus-sasl-2.1.23-15.el6_6.2.x86_64.rpm (cyrus-sasl-lib-2.1.23-15.el6_6.2.x86_64.rpm)
Using CRACK000.RPM;1 for  /myiso/Packages/cracklib-dicts-2.8.16-4.el6.x86_64.rpm (cracklib-2.8.16-4.el6.x86_64.rpm)
Using CRYPT000.RPM;1 for  /myiso/Packages/cryptsetup-luks-libs-1.2.0-11.el6.x86_64.rpm (cryptsetup-luks-1.2.0-11.el6.x86_64.rpm)
Using NCURS001.RPM;1 for  /myiso/Packages/ncurses-libs-5.7-4.20090207.el6.x86_64.rpm (ncurses-5.7-4.20090207.el6.x86_64.rpm)
Using DEVIC003.RPM;1 for  /myiso/Packages/device-mapper-multipath-0.4.9-106.el6.x86_64.rpm (device-mapper-libs-1.02.117-12.el6_9.1.x86_64.rpm)
Using NSS_S000.RPM;1 for  /myiso/Packages/nss-softokn-freebl-3.14.3-23.3.el6_8.x86_64.rpm (nss-softokn-3.14.3-23.3.el6_8.x86_64.rpm)
Using SELIN000.RPM;1 for  /myiso/Packages/selinux-policy-targeted-3.7.19-312.el6.noarch.rpm (selinux-policy-3.7.19-312.el6.noarch.rpm)
Using DEVIC004.RPM;1 for  /myiso/Packages/device-mapper-libs-1.02.117-12.el6_9.1.x86_64.rpm (device-mapper-persistent-data-0.6.2-0.2.rc7.el6.x86_64.rpm)
Using DEVIC005.RPM;1 for  /myiso/Packages/device-mapper-persistent-data-0.6.2-0.2.rc7.el6.x86_64.rpm (device-mapper-1.02.117-12.el6_9.1.x86_64.rpm)
Using PLYMO001.RPM;1 for  /myiso/Packages/plymouth-scripts-0.8.3-29.el6.centos.x86_64.rpm (plymouth-core-libs-0.8.3-29.el6.centos.x86_64.rpm)
Using OPENS000.RPM;1 for  /myiso/Packages/openssh-5.3p1-123.el6_9.x86_64.rpm (openssh-clients-5.3p1-123.el6_9.x86_64.rpm)
Using OPENS001.RPM;1 for  /myiso/Packages/openssh-clients-5.3p1-123.el6_9.x86_64.rpm (openssh-server-5.3p1-123.el6_9.x86_64.rpm)
Using E2FSP000.RPM;1 for  /myiso/Packages/e2fsprogs-libs-1.41.12-24.el6.x86_64.rpm (e2fsprogs-1.41.12-24.el6.x86_64.rpm)
Writing:   Initial Padblock                        Start Block 0
Done with: Initial Padblock                        Block(s)    16
Writing:   Primary Volume Descriptor               Start Block 16
Done with: Primary Volume Descriptor               Block(s)    1
Writing:   Eltorito Volume Descriptor              Start Block 17
Size of boot image is 4 sectors -> No emulation
Done with: Eltorito Volume Descriptor              Block(s)    1
Writing:   Joliet Volume Descriptor                Start Block 18
Done with: Joliet Volume Descriptor                Block(s)    1
Writing:   End Volume Descriptor                   Start Block 19
Done with: End Volume Descriptor                   Block(s)    1
Writing:   Version block                           Start Block 20
Done with: Version block                           Block(s)    1
Writing:   Path table                              Start Block 21
Done with: Path table                              Block(s)    4
Writing:   Joliet path table                       Start Block 25
Done with: Joliet path table                       Block(s)    4
Writing:   Directory tree                          Start Block 29
Done with: Directory tree                          Block(s)    31
Writing:   Joliet directory tree                   Start Block 60
Done with: Joliet directory tree                   Block(s)    22
Writing:   Directory tree cleanup                  Start Block 82
Done with: Directory tree cleanup                  Block(s)    0
Writing:   Extension record                        Start Block 82
Done with: Extension record                        Block(s)    1
Writing:   The File(s)                             Start Block 83
  2.18% done, estimate finish Sat Sep  5 23:23:59 2020
  4.36% done, estimate finish Sat Sep  5 23:23:59 2020
  6.53% done, estimate finish Sat Sep  5 23:23:59 2020
  8.71% done, estimate finish Sat Sep  5 23:23:59 2020
 10.89% done, estimate finish Sat Sep  5 23:23:59 2020
 13.06% done, estimate finish Sat Sep  5 23:23:59 2020
 15.24% done, estimate finish Sat Sep  5 23:23:59 2020
 17.41% done, estimate finish Sat Sep  5 23:23:59 2020
 19.59% done, estimate finish Sat Sep  5 23:23:59 2020
 21.77% done, estimate finish Sat Sep  5 23:23:59 2020
 23.94% done, estimate finish Sat Sep  5 23:24:03 2020
 26.12% done, estimate finish Sat Sep  5 23:24:02 2020
 28.29% done, estimate finish Sat Sep  5 23:24:02 2020
 30.47% done, estimate finish Sat Sep  5 23:24:02 2020
 32.65% done, estimate finish Sat Sep  5 23:24:02 2020
 34.82% done, estimate finish Sat Sep  5 23:24:01 2020
 37.00% done, estimate finish Sat Sep  5 23:24:01 2020
 39.18% done, estimate finish Sat Sep  5 23:24:01 2020
 41.36% done, estimate finish Sat Sep  5 23:24:01 2020
 43.53% done, estimate finish Sat Sep  5 23:24:01 2020
 45.71% done, estimate finish Sat Sep  5 23:24:01 2020
 47.88% done, estimate finish Sat Sep  5 23:24:01 2020
 50.06% done, estimate finish Sat Sep  5 23:24:00 2020
 52.24% done, estimate finish Sat Sep  5 23:24:00 2020
 54.41% done, estimate finish Sat Sep  5 23:24:00 2020
 56.59% done, estimate finish Sat Sep  5 23:24:00 2020
 58.77% done, estimate finish Sat Sep  5 23:24:02 2020
 60.95% done, estimate finish Sat Sep  5 23:24:02 2020
 63.12% done, estimate finish Sat Sep  5 23:24:02 2020
 65.30% done, estimate finish Sat Sep  5 23:24:02 2020
 67.47% done, estimate finish Sat Sep  5 23:24:01 2020
 69.65% done, estimate finish Sat Sep  5 23:24:01 2020
 71.82% done, estimate finish Sat Sep  5 23:24:01 2020
 74.00% done, estimate finish Sat Sep  5 23:24:01 2020
 76.18% done, estimate finish Sat Sep  5 23:24:01 2020
 78.36% done, estimate finish Sat Sep  5 23:24:01 2020
 80.53% done, estimate finish Sat Sep  5 23:24:01 2020
 82.71% done, estimate finish Sat Sep  5 23:24:01 2020
 84.88% done, estimate finish Sat Sep  5 23:24:01 2020
 87.06% done, estimate finish Sat Sep  5 23:24:01 2020
 89.24% done, estimate finish Sat Sep  5 23:24:02 2020
 91.41% done, estimate finish Sat Sep  5 23:24:02 2020
 93.59% done, estimate finish Sat Sep  5 23:24:02 2020
 95.77% done, estimate finish Sat Sep  5 23:24:02 2020
 97.94% done, estimate finish Sat Sep  5 23:24:02 2020
Total translation table size: 74272
Total rockridge attributes bytes: 33118
Total directory bytes: 59518
Path table size(bytes): 124
Done with: The File(s)                             Block(s)    229505
Writing:   Ending Padblock                         Start Block 229588
Done with: Ending Padblock                         Block(s)    150
Max brk space used 63000
229738 extents written (448 MB)
[root@localhost /]# ll /root/
total 459528
-r--r--r--. 1 root root     11862 Sep  5 23:21 34bae2d3c9c78e04ed2429923bc095005af1b166d1a354422c4c04274bae0f59-c6-minimal-x86_64.xml
-rw-------. 1 root root      1044 Sep  4 23:13 anaconda-ks.cfg
-rw-------. 1 root root      1044 Sep  4 23:16 anaconda-ks.cfg.bak
-rw-r--r--. 1 root root 470503424 Sep  5 23:24 CentOS-6.10_x86_64_auto_install.iso
-rw-r--r--. 1 root root      8901 Sep  4 23:13 install.log
-rw-r--r--. 1 root root      3384 Sep  4 23:13 install.log.syslog
-r--r--r--. 1 root root       924 Sep  4 23:27 isolinux.cfg
-rw-------. 1 root root      3859 Sep  5 23:00 ks6_minimal.cfg
-rwxr--r--. 1 root root       446 Sep  5 23:08 repo_create.sh
-rwxr--r--. 1 root root       139 Sep  5 23:14 yum_install.sh
8.MD5验证
[root@localhost /]# md5sum /root/CentOS-6.10_x86_64_auto_install.iso 
6e40ca423c007da034dda77da4638567  /root/CentOS-6.10_x86_64_auto_install.iso
[root@localhost /]# 
```

# 创建镜像

```shell
mkisofs -R -J -T -v --no-emul-boot --boot-load-size 4 --boot-info-table -V "CentOS-6.10-x86_64-minimal" -b isolinux/isolinux.bin -c isolinux/boot.cat -o /root/CentOS-6.10_x86_64_auto_install.iso /myiso/

mkisofs -R -J -T -v --no-emul-boot --boot-load-size 4 --boot-info-table -V "CentOS6-x86_64-minimal" -b isolinux/isolinux.bin -c isolinux/boot.cat -o /root/CentOS-6.10_x86_64_auto_install.iso /iso/
```