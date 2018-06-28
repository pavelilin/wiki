# CSF Firewall

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
