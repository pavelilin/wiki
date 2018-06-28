# WordPress

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

