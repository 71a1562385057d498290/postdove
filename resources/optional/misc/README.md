### Postfix - Logging - SMTP protocol errors

By default, **SMTP protocol errors** occurred during a session are not visible in the logs. This can be changed in a couple of ways described below.

1. By increasing the verbosity of the SMTP service:  
This will cause the errors to be logged to `syslog` or to the selected log file, along with a large amount of additional data.

2. By adding `protocol` to the list of `notify_classes`:  
This will cause a session transcript mail to be sent to the mailbox specified by the `error_notice_recipient` configuration parameter whenever a SMTP session contains protocol errors. By default the `error_notice_recipient` is `postmaster`.

        notify_classes = resource, software, protocol
        error_notice_recipient = postmaster

We can take the second method one step further and *inject* the session transcript mail into `syslog`.  
The steps required to do that are described below.

&nbsp;  
*/etc/postfix/main.cf*:
```
transport_maps = hash:/etc/postfix/smtpd_map_transport

...

notify_classes = resource, software, protocol
error_notice_recipient = logger@localhost
```

&nbsp;  
*/etc/postfix/master.cf*:
```
session_transcript unix  -       n       n       -       -       pipe
        flags=  user=nobody argv=/etc/postfix/session_transcript
```

&nbsp;  
*/etc/postfix/smtpd_map_transport*:
```
logger@localhost        session_transcript:
```

&nbsp;  
*/etc/postfix/session_transcript*:
```bash
#!/bin/bash
PATH=/bin:/usr/bin:/usr/local/bin:/usr/sbin

echo "$(cat)" | postlog -p info -t "postfix/session-transcript"
```