### Quota Status Policy Server Proxy

The QSPS Proxy sits between Postfix and Dovecot's quota-status policy server. Whenever Postfix performs a quota query for an alias the QSPS Proxy will translate that alias to a real mailbox, which is then used to query Dovecot's quota status policy server.

See the following mailing list thread for a loose description of the issue addressed by the QSPS Proxy:  
https://markmail.org/message/d3yicqklaszzcu65

**Note that this tool is not fully tested and it might not be production ready. Use it at your own risk.**

&nbsp;
### Limitations
This tool will not work with catch-all aliases (such as @domain.tld) or with aliases that point to multiple addresses.

&nbsp;
### Example configuration for using qsps-proxy

```
adduser --no-create-home --disabled-login --shell /usr/sbin/nologin qsps

mkdir /opt/qsps-proxy

cp qsps-proxy /opt/qsps-proxy
chmod 0550 /opt/qsps-proxy/qsps-proxy

chown -R qsps:qsps /opt/qsps-proxy

cp qsps-proxy.conf /etc
```

&nbsp;  
Enable the quota-status standalone network server.

*/etc/dovecot/dovecot.conf*:
```
    service quota-status {
        executable = quota-status -p postfix
        inet_listener {
            address = localhost
            port = 54325
        }
        client_limit = 1
    }
```

&nbsp;  
Create a new `spawn` service called `qsps-proxy` which will spawn the command specified by the `argv` parameter.  
You can read more about the `spawn` daemon here: http://www.postfix.org/spawn.8.html

*/etc/postfix/master.cf*:
```
    qsps-proxy unix    -    n    n    -    0   spawn
        user=qsps argv=/opt/qsps-proxy/qsps-proxy
```

&nbsp;  
Tell Postfix to use the `qsps-proxy` service by adding a `check_policy_service` restriction.

*/etc/postfix/main.cf*:
```
    smtpd_relay_restrictions =
        ...
        reject_unauth_destination,
        check_policy_service unix:private/qsps-proxy
```