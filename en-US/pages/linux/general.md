# Linux Tips & Tricks

## Prevent OOM killer

Edit file /etc/sysctl.conf

```bash
vm.overcommit_memory = 2 
vm.overcommit_ratio = 100
```

In case of bad memory usage (php out of memory) use this settings:

```bash
vm.overcommit_memory = 0
vm.overcommit_ratio = 80
```

## Who is using SWAP

```bash
grep VmSwap /proc/*/status 2>/dev/null | sort -nk2 | tail -n5
```

## Plesk on CentOS 12 bind fix

The problem was nginx was attempting to bind to port 443 before the IP was initialized.
To fix edit the /etx/sysctl.conf file and add:

```bash
net.ipv4.ip_nonlocal_bind = 1
```

## Find big files and folders

```bash
find / -mount -type f -print0 2>/dev/null | xargs -0 du 2>/dev/null | sort -n | tail -40 | cut -f2 | xargs -I{} du -sh 2>/dev/null {} | uniq; printf '+%.0s' {1..100}; echo; \
find / -mount -type d -print0 2>/dev/null | xargs -0 du 2>/dev/null | sort -n | tail -40 | cut -f2 | xargs -I{} du -sh 2>/dev/null {} | uniq; printf '+%.0s' {1..100}; echo; \
du -sh /var/cpanel/user_notifications && du -sh /backup/cpbackup/*/dirs/_var_cpanel/user_notifications
```

## Most accessed sites in the last minute

```bash
cat <<'SCRIPT' >>/root/sitesLoad.sh

#!/bin/bash
if [[ `netstat -ntalp | grep :80 | awk '$4 ~ /:80/ {print $0;exit}' | grep -q httpd; echo $?` -ne 0 ]]; then echo "Main web server is not Apache. Exiting..."; exit 1; fi

log=/tmp/hostPop
i=0
find /usr/local/apache/domlogs/ -type f -mmin -1 ! -group root -exec ls -l {} \+ | awk '{print $4, $9}' | column -t>$log

while read line; do
((++i))
       arr[$i]=$i
       arr[$i*1000]=$(printf "$line" | awk '{print $1}')
       arr[$i*1001]=$(printf "$line" | awk '{print $2}')
       arr[$i*1002]=$(wc -l `echo $line | awk '{print $NF}'` | cut -d' ' -f 1)
done < <(cat $log)

echo "Analyzing apache logs in realtime for 1 minute..."; sleep 60

for (( var=1 ; var<=$i ; var++ ))
do
       printf "${arr[$var*1000]} ${arr[$var*1001]} "
       echo $((`wc -l $(echo ${arr[$var*1001]}) | cut -d' ' -f 1` - ${arr[$var*1002]}));
done | sed -e 's/\/usr\/local\/apache\/domlogs\///g' | sort -nrk 3 | column -t

SCRIPT
chmod 700 /root/sitesLoad.sh && /root/sitesLoad.sh
```

## PHP.INI upload big files

```bash
ini_set('upload_max_filesize', '10M');
ini_set('post_max_size', '10M');
ini_set('max_input_time', 300);
ini_set('max_execution_time', 300);
```
## Change permissions (chmod) to folders and files

```bash
find . -type d -exec chmod 755 {} +                  
find . -type f -exec chmod 644 {} +    
```

## Disable IPv6

For current session:

```bash
echo 1 > /proc/sys/net/ipv6/conf/<interface-name>/disable_ipv6
echo 1 > /proc/sys/net/ipv6/conf/eth0/disable_ipv6
```
Permanent:

```bash
vi /etc/sysctl.conf
net.ipv6.conf.all.disable_ipv6 = 1
sudo sysctl -p /etc/sysctl.conf
```

## Virtual box boot from USB

```bash
VBoxManage internalcommands createrawvmdk -filename C:\usb.vmdk -rawdisk \\.\PhysicalDrive#
```

## Max connections on Linux

Add to /etc/sysctl.conf:

```bash
fs.file-max = 70000
net.ipv4.tcp_tw_recycle=0
net.ipv4.tcp_fin_timeout = 10
net.ipv4.ip_local_port_range = 15000 61000
net.core.somaxconn = 1024
net.core.netdev_max_backlog = 2000
```

## Rescan drives

```bash
echo "- - -" > /sys/class/scsi_host/host0/scan
echo 1 > /sys/class/scsi_device/2\:0\:0\:0/device/rescan
```

## Find PTR owner - reversal

```bash
dig 0.168.192.in-addr.arpa. NS
```

## Fix locales in Ubuntu

```bash
locale-gen en_US.UTF-8
dpkg-reconfigure locales
```

## Change permissions to all files and folders

```bash
chown `stat -c %U .`.`stat -c %U .` * -R
```

## Open last edited file

```bash
less `ls -dx1tr /usr/local/cpanel/logs/cpbackup/*|tail -1`
```

## Clear cache and swap

```bash
echo 3 > /proc/sys/vm/drop_caches && swapoff -a && swapon -a
```

## Resize root partition without reboot

Resize HD size in VC Run: (replace the device if needed)

```bash
echo 1 > /sys/class/scsi_device/2\:0\:0\:0/device/rescan
run fdisk:
```

print the old partition and save the output delete the root partition create a new partition using the SAME start block and make sure that the end block is higher than previous one (enter, enter, enter…) write changes (ignore error) run:

```bash
partx -u /dev/sda
resize2fs -f /dev/sda3
```

Make sure everything is OK:

```bash
df -h
```

## Delete files with a large file list - Argument list too long

```bash
find . -name '*'|xargs rm
```

## Kill process that runs more than X time

Kill cgi after 30 secs:

```bash
for i in `ps -eo pid,etime,cmd|grep cgi|awk '$2 > "00:30" {print $1}'`; do kill $i; done
```

## Who uses RAM

```bash
ps aux  | awk '{print $6/1024 " MB\t\t" $11}'  | sort -n
```

## Fix NFS mount on boot - Centos 7

Append text to the end of /usr/lib/systemd/system/nfs-idmap.service

```bash
[Install]
WantedBy=multi-user.target
```

Append text to the end of /usr/lib/systemd/system/nfs-lock.service

```bash
[Install]
WantedBy=nfs.target
```

Enable related services

```bash
systemctl enable nfs-idmapd.service 
systemctl enable rpc-statd.service 
systemctl enable rpcbind.socket
```

Reboot the server

## Update date and time on Linux server

```bash
sudo ntpd -qg; sudo hwclock -w
```

## Clone Linux user

```bash
#!/bin/bash
SRC=$1
DEST=$2

SRC_GROUPS=$(id -Gn ${SRC} | sed "s/${SRC} //g" | sed "s/ ${SRC}//g" | sed "s/ /,/g")
SRC_SHELL=$(awk -F : -v name=${SRC} '(name == $1) { print $7 }' /etc/passwd)

useradd --groups ${SRC_GROUPS} --shell ${SRC_SHELL} --create-home ${DEST}
passwd ${DEST}
```

## Clear BOOT on Ubuntu when 100%

```bash
dpkg --purge `dpkg --list|grep "linux-"|grep -v \`uname -r|sed 's/-generic//g'\`|cut -d" " -f3|grep "[0-9]-"|paste -sd " " -`
```

## Engintron generate custom_rules for all domains on the server.

```bash
ip=`hostname -i` && for domain in `httpd -S |  grep www. | awk -F 'www.' '{print $2}'`;do printf "if ( \$host ~ \"%s\") {set \$PROXY_DOMAIN_OR_IP \"$ip\";}\n" $domain ;done >> /etc/nginx/custom_rules && service nginx reload
```

## Fix Installatron - error 500 or missing list/install

```bash
/usr/local/installatron/repair -f --release --quick
```

[Installatron FIX](https://installatron.com/docs/admin/troubleshooting#missinginstalls)

## Find root of a site and CD to it

Replace domain.com with desired domain

```bash
cd `grep "domain.com" /etc/apache2/conf/httpd.conf -A7|grep Root|head -1|awk '{print $2}'`
```

## Generate CSR with OPENSSL

Generate key:

```bash
openssl genrsa -out private.key 2048
```

Gemnerate CSR:

```bash
openssl req -new -sha256 -key private.key -out mycsr.csr
```
