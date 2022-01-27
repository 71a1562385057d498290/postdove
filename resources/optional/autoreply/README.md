### Autoreplies

Setting up autoreplies is relatively easy but it does involve a bit of manual work.  

&nbsp;  
### Postfix example configuration for using autoreplies

Autoreplies are described in the following Postfix manual page: http://www.postfix.org/VIRTUAL_README.html#autoreplies  

First, tell Postfix where to look for `recipient address to transport` mappings. See the following manual pages for more details:  
http://www.postfix.org/postconf.5.html#transport_maps  
http://www.postfix.org/transport.5.html

*/etc/postfix/main.cf*:
```
    transport_maps = hash:/etc/postfix/smtpd_map_transport
```

&nbsp;    
Create a new pipe service called `autoreply` which will invoke the script specified by the `argv` parameter to handle delivery of autoreplies.  
You can read more about the pipe daemon here: http://www.postfix.org/pipe.8.html  

*/etc/postfix/master.cf*:
```
    autoreply unix  -       n       n       -       -       pipe
	    flags= user=nobody argv=/path/to/autoreply/script $sender $mailbox
```

&nbsp;  
Tell Postfix that mails for the `autoreply.domain.tld` domain will be handled by the `autoreply` service.

*/etc/postfix/smtpd_map_transport*:
```
    autoreply.domain.tld  autoreply:
```

&nbsp;  
Add a new virtual alias entry for the user for which you want the autoreply to be generated.  
Note that the way the virtual alias entry is created in the presented example will make sure that the mail is delivered to the recipient and a copy is sent to the address that produces automatic replies.

*/etc/postfix/smtpd_map_virtual_aliases*:
```
    user@domain.tld     user@domain.tld, user@domain.tld@autoreply.domain.tld
```

&nbsp;  
Write the script that will handle the autoreplies. A short autoreply sample script is listed below.

```bash
    #!/bin/bash
    PATH=/bin:/usr/bin:/usr/sbin

    to="${1}"
    from="${2}"

    sendmail -f "${from}" "${to}" << EOF
    Subject: Autoreply: Out of office
    From: ${from}

    This is an autoreply test!
    EOF
```

The presented script is for demonstration purposes only!

&nbsp;  
### Dovecot/Pigeonhole Sieve example configuration for using autoreplies

Another way of setting up autoreplies is by using Dovecot/Pigeonhole Sieve and the Vacation extension.  
More info is available at https://doc.dovecot.org/configuration_manual/sieve/usage/

First, we'll need to install the `dovecot-sieve` package and configure Dovecot to use the `sieve` plugin.
```
    apt install dovecot-sieve
```

*/etc/dovecot/dovecot.conf*:
```
    protocol lmtp {
        mail_plugins = $mail_plugins sieve
    }

    plugin {
        sieve = file:%h/sieve;active=%h/.dovecot.sieve
    }
```

Next, let's assume we want to setup an autoreply for user@domain.tld. We could create a script called `autoreply.sieve` for this. It could be as simple as the one presented below.

*autoreply.sieve*:
```
    require ["vacation"];
    
    vacation "Hey, I'm away on vacation. I'll read your message later.";
```

The presented script is for demonstration purposes only!

Next, create the mail user's `sieve` directory and place the `autoreply.sieve` scrip into it.  
Do not forget to properly set the ownership and permissions for the newly created `sieve` directory and its files.
```
    mkdir %h/sieve
    cp autoreply.sieve %h/sieve

    chown -R domainowner:domainowner %h/sieve
    chmod 0600 %h/sieve/autoreply.sieve
```

Note that **%h** is the mail user's home directory. It should be properly set up in your Dovecot configuration.
```
    mkdir /home/virtualmail/domain.tld/user/sieve
    cp autoreply.sieve /home/virtualmail/domain.tld/user/sieve

    chown -R domainowner:domainowner /home/virtualmail/domain.tld/user/sieve
    chmod 0600 /home/virtualmail/domain.tld/user/sieve/autoreply.sieve
```

A user can have multiple `sieve` scripts but only one of them can be active at a certain time.

To activate our `autoreply.sieve` script, we could issue the command listed below. Notice the lack of the `.sieve` file extension in the command!
```
    doveadm sieve activate -u <user@domain.tld> autoreply
```

To deactivate the currently active `sieve` script for the user@domain.tld mail user, simply run:
```
    doveadm sieve deactivate -u <user@domain.tld>
```