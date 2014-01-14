Add salt ping monitoring to zabbix
==================================

Create file "saltcmd" or copy from git

    vim /usr/local/etc/posts/saltcmd

Paste into "saltcmd" file

    #/bin/bash
    date +%s.%N > /usr/local/etc/posts/saltping
    salt '*' test.ping >> /usr/local/etc/posts/saltping
    date +%s.%N >> /usr/local/etc/posts/saltping

Add read and execute rights to "saltcmd" file

    chmod 555 /usr/local/etc/posts/saltcmd


Create file "saltping" or copy from git


    vim /usr/local/etc/posts/saltping

Add read and write rights to "saltping" file


    chmod 777 /usr/local/etc/posts/saltping


Add cron job to crontab


    crontab -e

Add crontab task to run "saltcmd" ("saltcmd" script runs every 10 minutes)


    */10 * * * * /usr/local/etc/posts/saltcmd


Edit zabbix-agent config.



    vim /etc/zabbix/zabbix_agentd.conf

Add to the end of file

    UserParameter=on.salt.ping.time,echo `tail -1 /usr/local/etc/posts/saltping` - `head -1 /usr/local/etc/posts/saltping` | bc|awk '{printf "%f", $0}'
    UserParameter=on.salt.ping.old,echo `date +%s.%N` - `head -1 /usr/local/etc/posts/saltping` | bc|awk '{printf "%f", $0}'
    UserParameter=on.salt.nodes,grep -c True$ /usr/local/etc/posts/saltping


Restart zabbix-agent to load new configurations.


    service zabbix-agent restart


Add template "Template_Saltstack.xml" found in git to zabbix and include it to monitoring host.

