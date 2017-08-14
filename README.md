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

## DKIM howto

See also https://kb.spamexperts.com/227853/.

```
mkdir /opt/dkim-keys
chmod 700 /opt/dkim-keys
pushd /opt/dkim-keys
openssl genrsa -out mailtrain.example.com.key 2048
openssl rsa -in mailtrain.example.com.key -out mailtrain.example.com.pub -pubout -outform PEM

#
# ### On your name server, add the DKIM (domainkey), DMARC and SPF records. make sure you
# ### know what each line does / what you are doing! The example shown below will
# ### - provide a DKIM public key to check if messages were really signed by your server
# ### - tell other mailservers to reject any mail from anything@mailtrain.example.com not sent
# ###   via mailtrain.example.com or mail.example.net (via SPF)
# ### - tell other mailservers to reject all messages that fail the DKIM signature check or
# ###   or are delivered via a host other then allowed in the SPF record.
# ### Basically even a small mistake in your DKIM/DMARC/SPF settings WILL cause your e-mails to never deliver!
#
#  default._domainkey.mailtrain.example.com     TXT    "k=rsa; p=[public key in one line];"
#  mailtrain.example.com                        TXT    "v=spf1 mx a a:mail.example.net -all"
#  _dmarc.mailtrain.example.com                 TXT    "v=DMARC1; p=reject"
#
# Configure mailtrain settings accoring to this:
#
# DKIM domain: mailtrain.example.com
# DKIM selector: default
# DKIM Private Key: [private key in /opt/dkim-keys/mailtrain.example.com.key]
#
# ZoneMTA will announce the public key presence under <DKIM selector>._domainkey.<DKIM domain>
# and will sign all mails with the private key.
#
# To test things out, use http://dkimvalidator.com (add one-time address to the subscribers)
#