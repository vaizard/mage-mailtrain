# mage-mailtrain

An Ansible role to install Mailtrain, following the official setup script https://github.com/Mailtrain-org/mailtrain/blob/master/setup/install.sh. Example playbook:

```yaml
---
# Need a random password? Try `openssl rand -base64 64`
- hosts: mailtrain
  vars:
      mailtrain_homepage: "https://example.com/"
      mailtrain_hostname: "mailtrain.example.com"
      mailtrain_mysql_database: mailtrain
      mailtrain_mysql_username: mailtrain
      mailtrain_mysql_password: "aHWwUvDRnDQ0X400811EA/j4DvTjtJwT+CiKoAuCUDoSUAyeoD9Jtcdd7mlBMqBq"
      mailtrain_mysql_username_ro: mailtrain_ro
      mailtrain_mysql_password_ro: "VKOqpsrkjKCWyo11kyxz45wVBm9MUmQpegvS5mFX1h2NliJPqcH0GF56ADfXMaYv"
      mailtrain_dkim_apikey: "hD559Spxx/oWF1101H4p42AUtGfQeEwFlEFbhg0PoBIfVn/IFDOk84GK1jbZcJ3+"
      mailtrain_smtp_password: "T85XKe6yjtIytdBwshBRQuvvzLNJUUeOfPKGnHLDeMZJP5Zz1/Rfao2Rbvgx1yoM"
      mailtrain_redis_password: "af6jXk/g5uOixN5IysdnAuDQNwaeYIKcaWH9ibmBJuz5o+FJEP5WgJHlvFAsWyHlD"
      mailtrain_default_sender: "My Example Company"
      mailtrain_default_postaddress:  "Example Street 123, Example City"
      mysql_root_password: "gfdm38HNnfmfkfkk331!_99nrnnr"
      redis_requirepass: "{{ mailtrain_redis_password }}"
      redis_enableppa: 'ppa:chris-lea/redis-server'
      mysql_databases:
       - name: "{{ mailtrain_mysql_database }}"
         encoding: utf8mb4
         collation: utf8mb4_unicode_ci
      mysql_users:
        - name: "{{ mailtrain_mysql_username }}"
          host: "127.0.0.1"
          password: "{{ mailtrain_mysql_password }}"
          priv: "{{ mailtrain_mysql_database }}.*:ALL"
        - name: "{{ mailtrain_mysql_username_ro }}"
          host: "127.0.0.1"
          password: "{{ mailtrain_mysql_password_ro }}"
          priv: "{{ mailtrain_mysql_database }}.*:SELECT"
        - name: "{{ mailtrain_mysql_username }}"
          host: "localhost"
          password: "{{ mailtrain_mysql_password }}"
          priv: "{{ mailtrain_mysql_database }}.*:ALL"
        - name: "{{ mailtrain_mysql_username_ro }}"
          host: "localhost"
          password: "{{ mailtrain_mysql_password_ro }}"
          priv: "{{ mailtrain_mysql_database }}.*:SELECT"
        - name: "{{ mailtrain_mysql_username }}"
          host: "127.0.1.1"
          password: "{{ mailtrain_mysql_password }}"
          priv: "{{ mailtrain_mysql_database }}.*:ALL"
        - name: "{{ mailtrain_mysql_username_ro }}"
          host: "127.0.1.1"
          password: "{{ mailtrain_mysql_password_ro }}"
          priv: "{{ mailtrain_mysql_database }}.*:SELECT"
      redis_unixsocket: '/run/redis/redis.sock'
      redis_pass: "{{ mailtrain_redis_pass }}"
      redis_maxmemory: 500mb
      redis_maxmemory_policy: "allkeys-lru"
  roles: 
    - role: mage-mysql
    - role: mage-mysql-backup
    - role: mage-redis
    - role: mage-mailtrain
```

## Set up MX, DKIM, DMARK, SPF and PTR

MX, DKIM, DMARK, SPF and PTR are DNS records which spam filters use to figure out if e-mails were
really sent by you (and not by a spammer who tries to conceal his identity to be able to continue
send bulks of e-mails people never subscribed for). Assuming that you use zone-mta and your
e-mails are to originate from a Mailtrain installation at `mailtrain.example.com` which is at IP
123.124.125.126 and optionally from `mail.example.net`, you will need

- the content of the files containing the private and public DKIM key this role generated for 
  you - both are to be found in the /opt/dkim-keys directory, mailtrain.example.com.key (private)
  mailtrain.example.com.pub (public) 
- add 3 new TXT records and 1 new MX record for the mailtrain.example.com. These will look most 
  likely quite similarly to the example below:

```
mailtrain.example.com                         MX    123.124.125.126
mailtrain.example.com                        TXT    "v=spf1 mx a a:mail.example.net -all"
default._domainkey.mailtrain.example.com     TXT    "k=rsa; p=[public key in one line];"
_dmarc.mailtrain.example.com                 TXT    "v=DMARC1; p=reject"
```

You can leave the DKIM entry as shown above, but refer to https://www.spfwizard.net/ and 
https://www.unlocktheinbox.com/dmarcwizard/ for better understanding of what the above means
and what values you might want to have in. You'll see that the above is a quite reasonable
example that you can (with a possible small tweak to the SPF record) use as is.

Now go to Mailtrain settings and set things up accoring to this:

**DKIM domain:** mailtrain.example.com
**DKIM selector:** default
**DKIM Private Key:** [copy and paste the private key in /opt/dkim-keys/mailtrain.example.com.key]

The above steps will have the following effect:

- all messages sent by Mailtrain / Zone-mta will be signed by the DKIM Private Key (the signature 
  becomes a part of the e-mail)
- when a spamfilter encounters this signature, it will look for the 
  **[DKIM selector]**._domainkey.**[DKIM domain]** TXT record, and use the public key stored there
  to verify that the signature is valid
- additionally, the spamfilter will look for a TXT SPF record and will look a if the e-mail was
  sent from the IP address of mailtrain.example.com or mail.example.net.  If the sender IP or
  domain is different, it will discard the e-mail as spam.
- furthermore, the spamfilter looks for the DMARC record, which tells it what to do with mails
  that aren't signed with DKIM or which don't have a valid signature. The example above will
  tell the spamfilter to reject such a mail as well.
- lastly, the spam filter will check if your sender domain has a MX record and if a SMTP server
  listens on any of the standard ports.

You are now almost set. To further confirm that you have full control over your network, the final
step is to set up a PTR record, which will give the right answer for a reverse DNS lookup (answer
to "what domain name is bound to IP address 123.124.125.126). If you run your own DNS, you probably
know it will look similar to this:

```
126.125.124.123.in-addr.arpa. 	1800 	PTR 	mailtrain.example.com.
```

If you run Mailtrain on a VPS, you will have to find the PTR configuration somewhere in your 
administration interface or ask your provider to help you.

## Testing

Use http://dkimvalidator.com (add one-time address to the subscribers of a testing list). When you
are happy, move on to https://www.mail-tester.com which will give you more details, but allows
only 3 tests per day for free.

