# Nginx

## Show all domains in NGINX configuration

```bash
grep -e "^\s*server_name" /etc/nginx/conf.d/*|sed -e 's/[\t ]*server_name//g;'|sed -e "s/ /\+/g"|sed -e 's/;//g'|while read line; do for i in $line; do echo -n "$i "|sed -e 's/://'  -e 's/\+/\n  |--/g'; done ;echo; done; echo
```