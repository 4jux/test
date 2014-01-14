Add salt ping monitoring to zabbix
==================================

Create file "saltcmd" or copy from git
<code>
vim /usr/local/etc/posts/saltcmd
</code>
Paste into file
<code>
#/bin/bash
date +%s.%N > /usr/local/etc/posts/saltping
salt '*' test.ping >> /usr/local/etc/posts/saltping
date +%s.%N >> /usr/local/etc/posts/saltping
</code>
Add 555 read and execute rights
<code>
chmod 555 /usr/local/etc/posts/saltcmd
</code>

Create file "saltping" or copy from git
<code>
vim /usr/local/etc/posts/saltping
</code>
Add 777 read and execute rights
<code>
chmod 777 /usr/local/etc/posts/saltping
</code>

Add cron job to crontab

<code>
crontab -e
#add (runs "saltcmd" script every 10 minutes)
*/10 * * * * /usr/local/etc/posts/saltcmd
</code>

Edit zabbix-agent config.

<code>
vim /etc/zabbix/zabbix_agentd.conf

#add to the end of file
UserParameter=on.salt.ping.time,echo `tail -1 /usr/local/etc/posts/saltping` - `head -1 /usr/local/etc/posts/saltping` | bc|awk '{printf "%f", $0}'
UserParameter=on.salt.ping.old,echo `date +%s.%N` - `head -1 /usr/local/etc/posts/saltping` | bc|awk '{printf "%f", $0}'
UserParameter=on.salt.nodes,grep -c True$ /usr/local/etc/posts/saltping
</code>

Restart zabbix-agent to load new configurations.
<code>
service zabbix-agent restart
</code>

Add template "Template_Saltstack.xml" found in git to zabbix and include it to monitoring host.

