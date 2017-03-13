# Base mail server role
This role configures postfix and dovecot to provide a basic mailserver with SMTP, IMAP and (ugh.) POP3

## How to use
Look into `defaults/main.yml` for settings to change. You also need a working LDAP and a MariaDB/MySQL database with a schema similar to `mail_mock`.

## Postfix
Postfix does users for the main domain directly against LDAP. Aliases are resolved via MariaDB. For the stored procedures, look into `mail_mock`. Authentication is done via SASL (Postfix can't do this via LDAP) and mail is delivered via LMTP. 
### Configuration specifics:
* Users need to authenticate to be able to send mail
* Authenticated users can only use their own adress and aliases which have the `can_send` flag set to 1 as a source adress. If you test this: Changing the from field in Thunderbird's compose view does not actually change the field, so don't get confused if it doesn't seem to work. You have to actually change the account's adress in the account settings view.
* TLS is required for clients. A self-signed certificate is copied iff no certificate is found

## Dovecot
Dovecot does auth using LDAP and additionally provides a SASL socket. Both sockets are created in the postfix chroot. POP3 does not actually delete mails, but only marks them as read and tags them with a special tag. All mails that are tagged like this are invisible on the next fetch, allowing people to use the same account with both POP3 and IMAP without loosing mails.

# TODO
## probably in separate roles (which this role maybe gets to depend on)
* Mailman (ideally mailman3) and dependencies
* spamassassin and dependencies
* unbound or kresd as validating resolvers, because this is important if a target server supports e.g. DANE

## probably in this role
* Dovecot sieve: Module does not yet get installed and config is not yet included but this is very useful, especially if we want to tag and sort mails from spamassassin
