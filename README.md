Nginx - Ansible Role
=========

[![Build Status](https://travis-ci.org/gbolo/ansible-role-nginx.svg?branch=master)](https://travis-ci.org/gbolo/ansible-role-nginx)

This role will fully configure and install [Nginx](http://nginx.org/) on RHEL/CentOS

Requirements
------------

- OS: CentOS 7
- Internet Connectivity

Role Variables
--------------

The following variables can be used to customize the docker installation:
```yaml
# OS RELATED -------------------------------------------------------------------

nginx_user: "nginx"
nginx_group: "nginx"
nginx_pkg_name: "nginx"
nginx_pkg_state: "present"
nginx_repo_enabled: true

nginx_service_name: "nginx"
nginx_service_enabled: true
nginx_service_state: "started"

# MAIN CONFIGURATION -----------------------------------------------------------

# nginx.conf
nginx_conf_template: "nginx.conf.j2"
nginx_pidfile: "/run/nginx.pid"
nginx_error_log: "/var/log/nginx/error.log warn"

nginx_worker_processes: "{{ ansible_processor_vcpus | default(ansible_processor_count) }}"
nginx_worker_connections: "1024"
nginx_multi_accept: "off"
nginx_access_log: "/var/log/nginx/access.log main buffer=16k"
nginx_sendfile: "on"
nginx_tcp_nopush: "on"
nginx_tcp_nodelay: "on"
nginx_keepalive_timeout: "65"
nginx_keepalive_requests: "100"
nginx_types_hash_max_size: "2048"
nginx_client_max_body_size: "64m"
nginx_server_names_hash_bucket_size: "64"
nginx_proxy_cache_path: ""
nginx_extra_http_options: ""

# hardening
nginx_hardening_enabled: true
nginx_server_tokens: "off"
nginx_ssl_protocols: "TLSv1.1 TLSv1.2"
nginx_ssl_ciphers: "EECDH+AESGCM:EDH+AESGCM:AES256+EECDH:AES256+EDH:!aNULL:!MD5"
nginx_ssl_session_cache: "shared:SSL:30m"
nginx_ssl_session_timeout: "5m"
nginx_header_hsts: "Strict-Transport-Security max-age=15768000"
nginx_ssl_dhparam: "/etc/nginx/tls/dhparam.pem"
nginx_ssl_dhparam_content: "!!ENCRYPT THIS WITH VAULT!!"

# conf.d
## !!WARNING!! due to this issue: https://github.com/ansible/ansible/issues/17159
## do not use null or booleans to exclude option because they will be turned into strings
nginx_confd:
  "01_default.conf":
    listen: "80 default_server"
    server_name: "example.com"
    root: "/var/www/example.com"
    index: "index.php index.html index.htm"
    access_log: "/var/log/nginx/default-access.log"
    error_log: "/var/log/nginx/default-error.log"
    extra_parameters: |
      location ~ \.php$ {
          fastcgi_split_path_info ^(.+\.php)(/.+)$;
          fastcgi_pass unix:/var/run/php5-fpm.sock;
          fastcgi_index index.php;
          fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
          include fastcgi_params;
      }
      location /protected {
          auth_basic            "Restricted";
          auth_basic_user_file  /etc/nginx/htpasswd/htpasswd1;
      }
  "02_proxy.conf":
    listen: "81"
    server_name: "example2.com"
    root: "/var/www/example.com"
    index: "index.php index.html index.htm"
    extra_parameters: |
      location / {
        proxy_pass              http://127.0.0.1:8080;
        proxy_set_header        Host            $host;
        proxy_set_header        X-Real-IP       $remote_addr;
        proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_connect_timeout   10s;
      }

# http basic auth
nginx_htpasswd_enabled: true
nginx_htpasswd_pkg_names:
  - "httpd-tools"
  - "python-passlib"

nginx_htpasswd:
  - path: "/etc/nginx/htpasswd/htpasswd1"
    users:
      - name: "user1"
        password: "!!ENCRYPT THIS WITH VAULT!!"
      - name: "user2"
        password: "!!ENCRYPT THIS WITH VAULT!!"
  - path: "/etc/nginx/htpasswd/htpasswd2"
    users:
      - name: "user3"
        password: "!!ENCRYPT THIS WITH VAULT!!"
```

Example Playbooks
----------------

Install hardened nginx with password protected web root
```
- hosts: localhost
  vars:
    nginx_hardening_enabled: true
    nginx_htpasswd_enabled: true
    nginx_confd:
      "01_default.conf":
        listen: "80 default_server"
        server_name: "example.com"
        root: "/usr/share/nginx/html"
        index: "index.html index.htm"
        access_log: "/var/log/nginx/default-access.log"
        error_log: "/var/log/nginx/default-error.log"
        nginx_htpasswd:
          - path: "/etc/nginx/htpasswd/default"
            users:
              - name: "user1"
                password: "secret"
              - name: "user2"
                password: "secret"
```
Install nginx with php fpm backend (php-fpm is not managed by this role)
```
- hosts: localhost
  vars:
    nginx_hardening_enabled: true
    nginx_htpasswd_enabled: false
    nginx_confd:
      "01_default.conf":
        listen: "80 default_server"
        server_name: "example.com"
        root: "/var/www/example.com"
        index: "index.php index.html index.htm"
        access_log: "/var/log/nginx/default-access.log"
        error_log: "/var/log/nginx/default-error.log"
        extra_parameters: |
          location ~ \.php$ {
              fastcgi_split_path_info ^(.+\.php)(/.+)$;
              fastcgi_pass unix:/var/run/php5-fpm.sock;
              fastcgi_index index.php;
              fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
              include fastcgi_params;
          }
```

TODO List
----------------
- add full support for https vhosts
- add support for stream blocks

Author and License
-------
`nginx` role written by:
- George Bolo | [linuxctl.com](https://linuxctl.com)

License: **MIT**

`FREE SOFTWARE, HELL YEAH!`
