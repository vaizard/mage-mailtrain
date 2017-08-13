# mage-mailtrain

An Ansible role to install Mailtrain, following the official setup script https://github.com/Mailtrain-org/mailtrain/blob/master/setup/install.sh. Example playbook:

```yaml
# Need a random password? Try `openssl rand -base64 64`
- hosts: mailtrain
  vars:
      mailtrain_homepage: "https://example.com/"
      mailtrain_hostname: "mailtrain.example.com"
      mailtrain_mysql_password: "aHWwUvDRnDQ0X4008JNEA/j4DvTjtJwT+CiKoAuCUDoSUAyeoD9Jtcdd7mlBMqBq"
      mailtrain_mysql_password_ro: "VKOqpsrkjKCWyoA/kyxz45wVBm9MUmQpegvS5mFX1h2NliJPqcH0GF56ADfXMaYv"
      mailtrain_dkim_apikey: "hD559Spxx/oWF1101H4p9xAUtGfQeEwFlEFbhg0PoBIfVn/IFDOk84GK1jbZcJ3+"
      mailtrain_smtp_password: "T85XKe6yjtIytdBwshBRQu98zLNJUUeOfPKGnHLDeMZJP5Zz1/Rfao2Rbvgx1yoM"
      mailtrain_redis_pass: "iN6jXk/g5uOixN5IysdnAuDQNwaeYIKcaWH9ibmBJuz5o+FJEP5WgJHlvFAsWyHlD"
      mysql_root_password: "9yqaN0DVfh68M//nqhhebd3BaywMganvprjLX6y+gQrR4gz8o75t3VzA2ZwC4qF+"
      mysql_databases:
       - name: "{{ mailtrain_mysql_database }}"
         encoding: utf8mb4
         collation: utf8mb4_unicode_ci
      mysql_users:
        - name: "{{ mailtrain_mysql_username }}"
          host: "localhost"
          password: "{{ mailtrain_mysql_password }}"
          priv: "{{ mailtrain_mysql_database }}.*:ALL"
        - name: "{{ mailtrain_mysql_username_ro }}"
          host: "localhost"
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
