# mage-mailtrain

An Ansible role to install Mailtrain, following the official setup script https://github.com/Mailtrain-org/mailtrain/blob/master/setup/install.sh. Example playbook:

```yaml
# Need a random password? Try `openssl rand -base64 64`
- hosts: mailtrain
  vars:
      mailtrain_hostname: "mailtrain.example.com"
      mailtrain_mysql_password: "VERYSECRETMYSQLPW"
      mailtrain_dkim_apikey: "VERYSECRETDKIMKEY"
      mailtrain_smtp_password: "VERYSECRETSMTPPW"
      mysql_root_password: "VERYSECRETMYSQLROOTPW"
      mysql_users:
        - name: {{ mailtrain_mysql_username }}
          host: "127.0.0.1"
          password: "{{ mailtrain_mysql_password }}"
          priv: "*.*:ALL"
      mysql_databases:
       - name: {{ mailtrain_mysql_database }}
         encoding: utf8mb4
         collation: utf8mb4_unicode_ci
      redis_unixsocket: '/run/redis/redis.sock'
      redis_pass: iN6jXk/g5uOixN5IydnAuDQNwaeYIKcaWH9ibmBJuz5o+FJEP5WgJHlvFAsWyHlDg8I38AmGOI5lN3qN/Etf/Q-q
      redis_maxmemory: 500mb
      redis_maxmemory_policy: "allkeys-lru"
  roles: 
    - role: mage-mysql
    - role: mage-redis    
    - role: mage-mailtrain
```
