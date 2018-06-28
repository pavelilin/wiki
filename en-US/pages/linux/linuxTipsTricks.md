# [Linux Tips & Tricks](https://github.com/fire1ce/wiki/blob/gh-pages/en-US/pages/linux/linuxTipsTricks.md)

## Wordpress login attack check

```bash
Crontab :
0 */6 * * *  sh /root/wplogin.sh

Script :
#!/bin/bash

###start editing
thold="100"
btime="359m"
###stop editing

egrep 'wp-login.php' /usr/local/apache/domlogs/* | grep -v ftp_log | awk -F : '{print $2}' | awk '{print $1}' | sort | uniq -c | sort -n | awk -v limit="$thold" '$1 > limit{print $2}' > $$_ip_$$

while IFS= read -r line
        do
                /usr/sbin/csf -td "$line"  "$btime" "banned for wordpress attack"
        done < $$_ip_$$
rm -f $$_ip_$$
```

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

### WHM replica

```bash
rsync -avz 192.168.1.122:/home/ /home/ --exclude="virtfs" --exclude="\.cp*" --exclude="cpeasyapache" 
rsync -avz 192.168.1.122:/usr/local/apache/conf/ /usr/local/apache/conf/
rsync -avz 192.168.1.122:/var/named/ /var/named/
rsync -avz 192.168.1.122:/usr/local/cpanel/ /usr/local/cpanel/
rsync -avz 192.168.1.122:/var/cpanel/ /var/cpanel
```

```bash
chkconfig cpanel off
chkconfig exim off
chkconfig dovecot off
chkconfig pure-ftpd off
chkconfig named off
chkconfig mysql off
chkconfig csf off
chkconfig iptables off
```

```bash
hostname: /etc/sysconfig/network
```

```bash
change shared ip - You can change it in /etc/wwwacct.conf infront of ADDR parameter.
change ip - Usage: /usr/local/cpanel/bin/setsiteip [-u user | domain] ip   (/etc/trueuserowners)
rebuild httpd conf
restart apache
```

## Apache memory usage

```bash
ps -ylC httpd | awk '{x += $8;y += 1} END {print "Apache Memory Usage (MB): "x/1024; print "Average Proccess Size (MB): "x/((y-1)*1024)}'
```

## Wordpress pingback

```bash
# WordPress Pingback Request Denial
if ($http_user_agent ~* "WordPress") {
  return 403;
}
```

Apache:
BrowserMatchNoCase WordPress wordpress_ping
BrowserMatchNoCase Wordpress wordpress_ping
Order Deny,Allow
Deny from env=wordpress_ping


## Find big files and folders

```bash
find / -mount -type f -print0 2>/dev/null | xargs -0 du 2>/dev/null | sort -n | tail -40 | cut -f2 | xargs -I{} du -sh 2>/dev/null {} | uniq; printf '+%.0s' {1..100}; echo; \
find / -mount -type d -print0 2>/dev/null | xargs -0 du 2>/dev/null | sort -n | tail -40 | cut -f2 | xargs -I{} du -sh 2>/dev/null {} | uniq; printf '+%.0s' {1..100}; echo; \
du -sh /var/cpanel/user_notifications && du -sh /backup/cpbackup/*/dirs/_var_cpanel/user_notifications
```

## Rebuild Sophos when disk full

```bash
/etc/init.d/postgresql92 rebuild
```

On older versions:


```bash
/var/mdw/scripts/smtp stop
dropdb -U postgres smtp
createdb -U postgres smtp
/var/mdw/scripts/smtp start
```
## Clear allowed networks on Sophos

Login to the Sophos

```bash
Type ‘cc’
In cc, you’ll be in MAIN, if not, type ‘MAIN’
Type ‘webadmin'
Type ‘allowed_networks@'
=['REF_NetworkAny']
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
## CSF GUI on ISPCONFIG 3

nstall old CSF (before 8.13)

Copy the ISPCONFIG folder to /etc/csf/ and enable CSF in ISPCONFIG

Backup csfui* files

Upgrade CSF

Copy backuped csfui* files back

Run the following commands:


```bash
sed -i 's/checkip/ConfigServer::CheckIP::checkip/g' /usr/local/csf/bin/csfui.pl
sed -i 's/sanity(/ConfigServer::Sanity::sanity(/g' /usr/local/csf/bin/csfui.pl
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
## CSF configuration

Automated configuration:

```bash
sed -i 's/TESTING = "1"/TESTING = "0"/' /etc/csf/csf.conf
sed -i 's/RESTRICT_SYSLOG = "0"/RESTRICT_SYSLOG = "3"/' /etc/csf/csf.conf
sed -i 's/IPV6 = "1"/IPV6 = "0"/' /etc/csf/csf.conf
sed -i 's/IGNORE_ALLOW = "0"/IGNORE_ALLOW = "1"/' /etc/csf/csf.conf
sed -i 's/CONNLIMIT = ""/CONNLIMIT = "22;5,80;70"/' /etc/csf/csf.conf
sed -i 's/PORTFLOOD = ""/PORTFLOOD = "22;tcp;5;300,80;tcp;100;1,443;tcp;100;5"/' /etc/csf/csf.conf
sed -i 's/CT_LIMIT = "0"/CT_LIMIT = "300"/' /etc/csf/csf.conf
```

Disable LFD alerts: Only if you want to disable them.

```bash
sed -i 's/LF_EMAIL_ALERT = "1"/LF_EMAIL_ALERT = "0"/' /etc/csf/csf.conf
sed -i 's/LF_SSH_EMAIL_ALERT = "1"/LF_SSH_EMAIL_ALERT = "0"/' /etc/csf/csf.conf
sed -i 's/LF_SU_EMAIL_ALERT = "1"/LF_SU_EMAIL_ALERT = "0"/' /etc/csf/csf.conf
sed -i 's/LF_WEBMIN_EMAIL_ALERT = "1"/LF_WEBMIN_EMAIL_ALERT = "0"/' /etc/csf/csf.conf
sed -i 's/LF_CONSOLE_EMAIL_ALERT = "1"/LF_CONSOLE_EMAIL_ALERT = "0"/' /etc/csf/csf.conf
sed -i 's/LT_EMAIL_ALERT = "1"/LT_EMAIL_ALERT = "0"/' /etc/csf/csf.conf
sed -i 's/CT_EMAIL_ALERT = "1"/CT_EMAIL_ALERT = "0"/' /etc/csf/csf.conf
sed -i 's/PS_EMAIL_ALERT = "1"/PS_EMAIL_ALERT = "0"/' /etc/csf/csf.conf
```

Disable PT alerts: Only if you want to disable them.

```bash
sed -i 's/PT_USERPROC = "10"/PT_USERPROC = "0"/' /etc/csf/csf.conf
sed -i 's/PT_USERMEM = "256"/PT_USERMEM = "0"/' /etc/csf/csf.conf
sed -i 's/PT_USERTIME = "1800"/PT_USERTIME = "0"/' /etc/csf/csf.conf
sed -i 's/PT_USERKILL_ALERT = "1"/PT_USERKILL_ALERT = "0"/' /etc/csf/csf.conf
sed -i 's/PT_LOAD = "30"/PT_LOAD = "0"/' /etc/csf/csf.conf
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
```
```bash
```
```bash
```
```bash
```
```bash
```
```bash
```
```bash
```
```bash
```
```bash
```
```bash
```
```bash
```
```bash
```
```bash
```
```bash
```
```bash
```
```bash
```
```bash
```
```bash
```
```bash
```
```bash
```
```bash
```
