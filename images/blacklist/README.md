## DNS blacklist for spamassassin

This is based on [Running Your Own RBL DNS
Blacklist](http://www.blue-quartz.com/rbl/) using the Debian rbldnsd
package adapted from scripts published by Herb Rubin some years
ago. This attempts to counter large-scale botnets (with hundreds of
thousands of scattered IP addresses) that spammers use to bypass the
well-known DNSBL sites. We do this by examining known-spam messages'
Received headers, and inserting their source IP addresses into a local
MySQL table, almost immediately blacklisting against multiple uses of
any given source IP. This tool maintains that table and provides a
local DNSBL which you can add to Spamassassin's rules. To make the
most use of it, set up honeypot email addresses separate from your
primary users' addresses; greylisting unknown senders for several
minutes will add to this protection.

Before running it, grant access to a mysql user thus:

    USER=blacklister
    PSWD=xxx
    mysql> GRANT SELECT,UPDATE,INSERT,CREATE ON `blacklist`.* TO
      '$USER'@'10.%' IDENTIFIED BY '$PSWD';

Add a mysql-blacklist-user that contains the $PSWD you've set:

    # docker secret create mysql-blacklist -
    user=blacklister
    password=$PSWD

Decide on a subdomain name, such as blacklist.yourdomain.com. See
swarm-stack.yml for an example; set that name as an environment variable
RBL_DOMAIN. To delegate to this subdomain, list hosts where you'll
be running this in environment variable NS_SERVERS (if you're running
a swarm cluster, this will be the DNS names of the cluster nodes).

Then to add new IPs into the blacklist, set up procmail to run the
honeypot-ip.py parser script (included here under src directory) to
insert into the MySQL ips table upon receipt of any known spam message.
Example:

  :0fw
  #| /usr/local/bin/honeypot-ip.py --db-config ~/.my.cnf \
    --honeypot honeyforbees@instantlinux.net \
    --relay 'by mx-caprica.?\.easydns\.com'

Add a .my.cnf file with db credentials:

    [client]
    host=xdb00
    database=blacklist
    user=blacklist
    password=xxx

This script can also be invoked as a spamfilter under postfix; use
the --pipe-stdout command option for that use case.
