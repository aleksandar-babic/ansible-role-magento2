# Ansible role for Magento 2 CE LEMP+Redis stack setup and installation
> Role is tested on RHEL/CentOS 7, it should work with RHEL/CentOS 6 but it's not guaranteed.
**This role is meant to be used on fresh instances of RHEL/CentOS, it shouldn't be used if you already have some of packages installed.**
### Overview of role tasks
* Install and configure NGINX
* Install and configure php-fpm 7.0
* Install and add to path Composer
* Create Magento 2 CE project with composer
* Setup NGINX vhosts as per magento dev docs recommendations
* Install and configure MariaDB
* Install and configure Redis In-Memory DB
* Install Magento 2 CE using CLI
* Setup Magento 2 Cron jobs
* TODO : Configure env.php to use redis

## Example of playbook that is using this role
#### File structure
![file_structure](https://i.imgur.com/QIQwS8o.png)
#### Playbook (example.yml)
```yaml
---
- hosts: magentoservers
  roles:
    - role: mage2
      nginx_vhosts:
        - listen: "80"
          server_name: "example.com"
          mage_root: "/var/www/html"
      nginx_upstreams:
        - name: fastcgi_backend
          servers: {
            "127.0.0.1:9000"
          }    
      magento_root_dir: "/var/www/html"
      magento_account_public_key: "YOUR_PUBLIC_KEY"
      magento_account_private_key: "YOUR_PRIVATE_KEY"     
      mariadb_bind_address: '127.0.0.1'
      mariadb_root_password: 'supersupersecret'
      mariadb_databases:
        - name: mage2db
      mariadb_users:
        - name: mage2usr
          password: 'supersecret@'
          priv: "mage2db.*:ALL"
      magento_mysql_user: "mage2usr"
      magento_mysql_password: "supersecret@"
      magento_mysql_dbname: "mage2db"
      magento_admin_user: "admin"
      magento_admin_password: "adminadmin!1"
      magento_admin_email: "admin@admin.com"
      magento_admin_firstname: "Admin"
      magento_admin_lastname: "Adminer"
      magento_language: "en_US"
      magento_currency: "USD"
      magento_base_url: "http://example.com"
```

## Can I change some of parameters (like memory_limit, etc..) ?
Yes, you can override any of values from defaults/main.yml, just add it to your playbook (exactly the same as I overrided for example ```magento_base_url``` in example playbook).
```yaml
---

###
# NGINX Default variables
###

# Used only for Redhat installation, enables source Nginx repo.
nginx_yum_repo_enabled: true

# The name of the nginx package to install.
nginx_package_name: "nginx"

nginx_conf_template: "nginx.conf.j2"
nginx_vhost_template: "vhost.j2"

nginx_worker_processes: "{{ ansible_processor_vcpus | default(ansible_processor_count) }}"
nginx_worker_connections: "1024"
nginx_multi_accept: "off"

nginx_error_log: "/var/log/nginx/error.log warn"
nginx_access_log: "/var/log/nginx/access.log main buffer=16k flush=2m"

nginx_sendfile: "on"
nginx_tcp_nopush: "on"
nginx_tcp_nodelay: "on"

nginx_keepalive_timeout: "65"
nginx_keepalive_requests: "100"

nginx_server_tokens: "on"

nginx_client_max_body_size: "64m"

nginx_server_names_hash_bucket_size: "64"

nginx_proxy_cache_path: ""

nginx_extra_conf_options: ""
# Example extra main options, used within the main nginx's context:
#   nginx_extra_conf_options: |
#     env VARIABLE;
#     include /etc/nginx/main.d/*.conf;

nginx_extra_http_options: ""
# Example extra http options, printed inside the main server http config:
#    nginx_extra_http_options: |
#      proxy_buffering    off;
#      proxy_set_header   X-Real-IP $remote_addr;
#      proxy_set_header   X-Scheme $scheme;
#      proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
#      proxy_set_header   Host $http_host;

nginx_remove_default_vhost: false
nginx_vhosts: []
# Example vhost below, showing all available options:
# - listen: "80" # default: "80"
#   server_name: "example.com" # default: N/A
#   filename: "example.com.conf" # Can be used to set the filename of the vhost file.
#
#   # Properties that are only added if defined:
#   error_page: ""
#   access_log: ""
#   error_log: ""
#   mage_root: ""
#   extra_parameters: "" # Can be used to add extra config blocks (multiline).
#   template: "" # Can be used to override the `nginx_vhost_template` per host.
#   state: "absent" # To remove the vhost configuration.

nginx_upstreams: []
# - name: myapp1
#   strategy: "ip_hash" # "least_conn", etc.
#   keepalive: 16 # optional
#   servers: {
#     "srv1.example.com",
#     "srv2.example.com weight=3",
#     "srv3.example.com"
#   }

nginx_log_format: |
  '$remote_addr - $remote_user [$time_local] "$request" '
  '$status $body_bytes_sent "$http_referer" '
  '"$http_user_agent" "$http_x_forwarded_for"'

###
# PHP-FPM Default variables
###

php_fpm_template: "php-fpm.j2"

php_fpm_pool_name: "www" 
php_fpm_user: "nginx" 
php_fpm_group: "nginx"
php_fpm_ip_port: "127.0.0.1:9000" 

php_max_execution_time: 1800
php_memory_limit: "2G"
php_timezone: "America/Phoenix"

###
# Magento2 Default variables
###

magento_account_public_key: ""
magento_account_private_key: ""
magento_root_dir: ""
magento_admin_frontname: "admin"
magento_mysql_user: ""
magento_mysql_password: ""
magento_mysql_dbname: ""
magento_admin_user: ""
magento_admin_password: ""
magento_admin_email: ""
magento_admin_firstname: ""
magento_admin_lastname: ""
magento_language: "en_US"
magento_currency: "USD"
magento_base_url: ""


###
# MariaDB Default variables
###

mariadb_databases: []
mariadb_users: []
mariadb_root_password: ''

mariadb_version: '10.2'

mariadb_swappiness: 0

# Network configuration (network.cnf)
mariadb_bind_address: '127.0.0.1'

# Server configuration (server.cnf)
mariadb_port: 3306
mariadb_skip_name_resolve: 1

mariadb_log_warnings: '1'
mariadb_slow_query_log: '0'
mariadb_long_query_time: '10'

mariadb_max_connections: 505
mariadb_max_user_connections: 500
mariadb_max_allowed_packet: '16M'
mariadb_max_heap_table_size: '16M'
ansible_wait_timeout: 60

mariadb_query_cache_size: 0
mariadb_sort_buffer_size: '2M'
mariadb_tmp_table_size: '16M'
mariadb_read_buffer_size: '128k'
mariadb_read_rnd_buffer_size: '256k'
mariadb_join_buffer_size: '128k'
mariadb_table_definition_cache: '1400'
mariadb_table_open_cache: '2000'
mariadb_table_open_cache_instances: '8'

mariadb_innodb_strict_mode: 'ON'
mariadb_innodb_file_format_check: 1
mariadb_innodb_file_format: Barracuda
mariadb_innodb_buffer_pool_size: '384M'
mariadb_innodb_buffer_pool_instances: 8
mariadb_innodb_file_per_table: 'ON'
mariadb_innodb_flush_log_at_trx_commit: 1
mariadb_innodb_log_buffer_size: '16M'
mariadb_innodb_log_file_size: '48M'

###
# Redis Default variables
###
redis_template: "redis.conf.j2"
redis_config_name: "redis.conf"
redis_bind_ip: 127.0.0.1
redis_port: 6379
```