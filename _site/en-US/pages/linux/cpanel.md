# cPanel & WHM

## WHM replica

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

## Disable core files in CPanel account

Add this in /etc/sysctl.conf

```bash
kernel.core_uses_pid = 0
kernel.core_pattern = /dev/null
```

And then run:

```bash
sysctl -p
```

## Add HSTS support in CPANEL

```bash
cp -p /var/cpanel/templates/apache2_4/ssl_vhost.default /var/cpanel/templates/apache2_4/ssl_vhost.local
vi /var/cpanel/templates/apache2_4/ssl_vhost.local
```

Edit:

```bash
<VirtualHost[% FOREACH ipblock IN vhost.ips %] [% ipblock.ip %]:[% ipblock.port %][% END %]>
 # Enable HTTP Strict Transport Security
 Header always set Strict-Transport-Security "max-age=63072000; includeSubdomains;"
```

```bash
cp -p /var/cpanel/templates/apache2_4/main.default /var/cpanel/templates/apache2_4/main.local
vi /var/cpanel/templates/apache2_4/main.local
```

Edit:

```bash
[% IF main.sslprotocol.item.sslprotocol.length %]SSLProtocol [% main.sslprotocol.item.sslprotocol %][% END %]
SSLHonorCipherOrder on
```

Run:

```bash
/scripts/rebuildhttpdconf
service httpd restart
```
