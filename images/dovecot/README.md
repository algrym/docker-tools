## dovecot

The dovecot imapd daemon in a small Alpine Linux container, with
postfix for local delivery.

### Usage

Configuration is defined as files in a volume mounted as
/etc/dovecot/conf.local. Within that directory:

* Define your local settings as dovecot.conf.

* If you have an LDAP server, put its settings in dovecot-ldap.conf.

Also configure postfix as described in the postfix image.

See etc-example directory and docker-compose.yml.

### Variables

| Variable | Default | Description |
| -------- | ------- | ----------- |
| LDAP_PASSWD_SECRET | ldap-ro-passwd | name of secret for LDAP credential |
| TZ | US/Pacific | time zone |

### Secrets

| Secret | Description |
| ------ | ----------- |
| ldap-ro-passwd | password for looking up LDAP users |
