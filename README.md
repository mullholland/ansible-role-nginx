# [nginx](#nginx)

installs and configures nginx

|GitHub|GitLab|Quality|Downloads|Version|
|------|------|-------|---------|-------|
|[![github](https://github.com/mullholland/ansible-role-nginx/workflows/Ansible%20Molecule/badge.svg)](https://github.com/mullholland/ansible-role-nginx/actions)|[![gitlab](https://gitlab.com/opensourceunicorn/ansible-role-nginx/badges/master/pipeline.svg)](https://gitlab.com/opensourceunicorn/ansible-role-nginx)|[![quality](https://img.shields.io/ansible/quality/58676)](https://galaxy.ansible.com/mullholland/nginx)|[![downloads](https://img.shields.io/ansible/role/d/58676)](https://galaxy.ansible.com/mullholland/nginx)|[![Version](https://img.shields.io/github/release/mullholland/ansible-role-nginx.svg)](https://github.com/mullholland/ansible-role-nginx/releases/)|

## [Example Playbook](#example-playbook)

This example is taken from [`molecule/default/converge.yml`](https://github.com/mullholland/ansible-role-nginx/blob/master/molecule/default/converge.yml) and is tested on each push, pull request and release.

```yaml
---
- name: Converge
  hosts: all
  become: true
  gather_facts: true
  vars:
    nginx_repo_version: "mainline"
    nginx_custom_log_formats:
      - name: "kv"
        format: |-
          'site="$server_name" server="$host" dest_port="$server_port" dest_ip="$server_addr" '
          'src="$remote_addr" src_ip="$realip_remote_addr" user="$remote_user" '
          'time_local="$time_local" protocol="$server_protocol" status="$status" '
          'bytes_out="$bytes_sent" bytes_in="$upstream_bytes_received" '
          'http_referer="$http_referer" http_user_agent="$http_user_agent" '
          'nginx_version="$nginx_version" http_x_forwarded_for="$http_x_forwarded_for" '
          'http_x_header="$http_x_header" uri_query="$query_string" uri_path="$uri" '
          'http_method="$request_method" response_time="$upstream_response_time" '
          'cookie="$http_cookie" request_time="$request_time" category="$sent_http_content_type" https="$https"'

    nginx_vhosts:
      - listen: "80 default_server"
        server_name: "_"
        extra_parameters: |
          return 301 https://$host$request_uri;

    nginx_streams:
      - listen: "66 udp"
        proxy_pass: dns_servers
        extra_parameters: |
          proxy_responses 0;
    nginx_upstreams:
      - name: dns_servers
        servers:
          - "192.168.136.130:53"
          - "192.168.136.131:53 weight=3"
      - name: myapp
        strategy: "hash $remote_addr consistent"
        servers:
          - "192.168.136.11:8080"
          - "192.168.136.12:8080"
          - "192.168.136.13:8080"


  roles:
    - role: "mullholland.nginx"

    # nginx_stream_server:
    #   - listen: "53 udp"
    #     proxy_pass: "dns_servers"
    # nginx_stream_upstreams:
    #   - name: "dns_servers"
    #     strategy: "hash"
    #     servers:
    #      - "192.168.136.130:53"
    #      - "192.168.136.131:53"

    # nginx_http_upstreams:
    #   - name: myapp
    #     strategy: "hash"
    #     servers:
    #       - "192.168.136.11:8080"
    #       - "192.168.136.12:8080"
    #       - "192.168.136.13:8080"
```

The machine needs to be prepared. In CI this is done using [`molecule/default/prepare.yml`](https://github.com/mullholland/ansible-role-nginx/blob/master/molecule/default/prepare.yml):

```yaml
---
- name: Prepare
  hosts: all
  become: true
  gather_facts: true

  tasks:
    - name: Update apt cache
      ansible.builtin.apt:
        update_cache: true
      when:
        - ansible_os_family == "Debian"
```


## [Role Variables](#role-variables)

The default values for the variables are set in [`defaults/main.yml`](https://github.com/mullholland/ansible-role-nginx/blob/master/defaults/main.yml):

```yaml
---
# ---------------------------------------------------------------------------
# nginx repository
# ---------------------------------------------------------------------------
# Use the official Nginx Repositories(stable/mainline)
# https://www.nginx.com/blog/nginx-1-6-1-7-released/
# Note that stable does not mean more reliable or more bug-free.
# In fact, the mainline is generally regarded as more reliable
# because we port all bug fixes to it, and not just critical fixes
# as for the stable branch. On the other hand, changes in the
# stable branch are very unlikely to affect third-party modules.
# We don’t make the same commitment concerning the mainline,
# where new features can affect the operation of third-party
# modules.
#
# We recommend that in general you deploy the NGINX mainline
# branch at all times.
nginx_repo_version: "mainline"
nginx_signing_key: "https://nginx.org/keys/nginx_signing.key"

# The name of the nginx apt/yum package to install.
nginx_package_name: "nginx"

# ---------------------------------------------------------------------------
# nginx.conf
# ---------------------------------------------------------------------------
nginx_worker_processes: "auto"
nginx_worker_connections: "1024"
nginx_multi_accept: "off"

nginx_error_log: "/var/log/nginx/error.log notice"
nginx_access_log: "/var/log/nginx/access.log main buffer=16k"

nginx_sendfile: "on"
nginx_tcp_nopush: "on"
nginx_tcp_nodelay: "on"

nginx_keepalive_timeout: "65"
nginx_keepalive_requests: "100"

nginx_client_max_body_size: "64m"

nginx_server_names_hash_bucket_size: "64"

nginx_proxy_cache_path: ""

# set custom log_format
nginx_custom_log_formats: []
# nginx_custom_log_formats:
#   - name: "kv"
#     format: |-
#       'site="$server_name" server="$host" dest_port="$server_port" dest_ip="$server_addr" '
#       'src="$remote_addr" src_ip="$realip_remote_addr" user="$remote_user" '
#       'time_local="$time_local" protocol="$server_protocol" status="$status" '
#       'bytes_out="$bytes_sent" bytes_in="$upstream_bytes_received" '
#       'http_referer="$http_referer" http_user_agent="$http_user_agent" '
#       'nginx_version="$nginx_version" http_x_forwarded_for="$http_x_forwarded_for" '
#       'http_x_header="$http_x_header" uri_query="$query_string" uri_path="$uri" '
#       'http_method="$request_method" response_time="$upstream_response_time" '
#       'cookie="$http_cookie" request_time="$request_time" category="$sent_http_content_type" https="$https"'
#   - name: "json_analytics"
#     format: |-
#       escape=json '{'
#       '"msec": "$msec", ' # request unixtime in seconds with a milliseconds resolution
#       '"connection": "$connection", ' # connection serial number
#       '"connection_requests": "$connection_requests", ' # number of requests made in connection
#       '"pid": "$pid", ' # process pid
#       '"request_id": "$request_id", ' # the unique request id
#       '"request_length": "$request_length", ' # request length (including headers and body)
#       '"remote_addr": "$remote_addr", ' # client IP
#       '"remote_user": "$remote_user", ' # client HTTP username
#       '"remote_port": "$remote_port", ' # client port
#       '"time_local": "$time_local", '
#       '"time_iso8601": "$time_iso8601", ' # local time in the ISO 8601 standard format
#       '"request": "$request", ' # full path no arguments if the request
#       '"request_uri": "$request_uri", ' # full path and arguments if the request
#       '"args": "$args", ' # args
#       '"status": "$status", ' # response status code
#       '"body_bytes_sent": "$body_bytes_sent", ' # the number of body bytes exclude headers sent to a client
#       '"bytes_sent": "$bytes_sent", ' # the number of bytes sent to a client
#       '"http_referer": "$http_referer", ' # HTTP referer
#       '"http_user_agent": "$http_user_agent", ' # user agent
#       '"http_x_forwarded_for": "$http_x_forwarded_for", ' # http_x_forwarded_for
#       '"http_host": "$http_host", ' # the request Host: header
#       '"server_name": "$server_name", ' # the name of the vhost serving the request
#       '"request_time": "$request_time", ' # request processing time in seconds with msec resolution
#       '"upstream": "$upstream_addr", ' # upstream backend server for proxied requests
#       '"upstream_connect_time": "$upstream_connect_time", ' # upstream handshake time incl. TLS
#       '"upstream_header_time": "$upstream_header_time", ' # time spent receiving upstream headers
#       '"upstream_response_time": "$upstream_response_time", ' # time spend receiving upstream body
#       '"upstream_response_length": "$upstream_response_length", ' # upstream response length
#       '"upstream_cache_status": "$upstream_cache_status", ' # cache HIT/MISS where applicable
#       '"ssl_protocol": "$ssl_protocol", ' # TLS protocol
#       '"ssl_cipher": "$ssl_cipher", ' # TLS cipher
#       '"scheme": "$scheme", ' # http or https
#       '"request_method": "$request_method", ' # request method
#       '"server_protocol": "$server_protocol", ' # request protocol, like HTTP/1.1 or HTTP/2.0
#       '"pipe": "$pipe", ' # "p" if request was pipelined, "." otherwise
#       '"gzip_ratio": "$gzip_ratio", '
#       '"http_cf_ray": "$http_cf_ray",'
#       '"geoip_country_code": "$geoip_country_code"'
#       '}'

# Example extra main options, used within the main nginx's context:
nginx_conf_extra_options: []
# nginx_conf_extra_options: |
#     env VARIABLE;
#     include /etc/nginx/main.d/*.conf;

# append http options
nginx_conf_extra_http_options: ""
# nginx_conf_extra_http_options: |
#   proxy_buffering    off;
#   proxy_set_header   X-Real-IP $remote_addr;
#   proxy_set_header   X-Scheme $scheme;
#   proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
#   proxy_set_header   Host $http_host;

# to append http options set in all by specific group_vars
nginx_conf_extra_http_options_group: ""
# nginx_conf_extra_http_options_group: |
#   proxy_buffering    off;
#   proxy_set_header   X-Real-IP $remote_addr;
#   proxy_set_header   X-Scheme $scheme;
#   proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
#   proxy_set_header   Host $http_host;


# ---------------------------------------------------------------------------
# nginx vhosts
# ---------------------------------------------------------------------------
# Example vhost below, showing all available options:
nginx_vhosts:
  #   # required => default: "80 default_server"
  # - listen: "80 default_server"
  #   # optional, only added if defined
  #   server_name: "_"
  #   # optional, only added if defined
  #   root: "/var/www/example.com"  # default: N/A
  #   # optional => default: "index.html index.htm"
  #   index: "index.html index.htm"
  #   # optional, only added if defined
  #   error_page: ""
  #   # optional, only added if defined
  #   access_log: ""
  #   # optional, only added if defined
  #   error_log: ""
  #   # optional, only added if defined
  #   # Can be used to add extra config blocks (multiline).
  #   extra_parameters: ""
  #   # extra_parameters: |
  #   #   ssl_certificate     www.example.com.crt;
  #   #   ssl_certificate_key www.example.com.key;
  #   #   ssl_protocols       TLSv1 TLSv1.1 TLSv1.2;
  #   #   ssl_ciphers         HIGH:!aNULL:!MD5;
  #
  # Redirect All Sites to HTTPS
  - listen: "80 default_server"
    server_name: "_"
    extra_parameters: |
      return 301 https://$host$request_uri;

# ---------------------------------------------------------------------------
# nginx streams
# ---------------------------------------------------------------------------

# Configuring TCP or UDP Load Balancing
#
# If you are configuring Nginx as a load balancer,
# you can define one or more upstream sets using this
# variable. In addition to defining at least one upstream,
# you would need to configure one of your server blocks to
# proxy requests through the defined upstream
# (e.g. `proxy_pass http://myapp1;`). See the
# commented example in `defaults/main.yml` for
# more information.
nginx_upstreams: []
# nginx_upstreams:
#   # Required. Name of the upstream
#   - name: dns_servers
#     # Required. Server to balance to
#     servers:
#       - "192.168.136.130:53"
#       - "192.168.136.131:53 weight=3"
#   # Required. Name of the upstream
#   - name: myapp
#     # Optional. load‑balancing method, if not set round-robin will be used
#     strategy: "hash $remote_addr consistent"
#     # Required. Server to balance to
#     servers:
#       - "192.168.136.11:8080"
#       - "192.168.136.12:8080"
#       - "192.168.136.13:8080"

# Configuring Reverse Proxy
#
# If you are configuring Nginx as a tcp/udp streaming
# load balancer, list the stream definitions here.
# https://docs.nginx.com/nginx/admin-guide/load-balancer/tcp-udp-load-balancer/
# An example of simple udp stream of port 53 to a upstream
# group of dns servers.
nginx_streams: []
# nginx_streams:
#   # Required. Port or IP:PORT and protocol
#   - listen: "66 udp"
#     # Required. Destination server:port or upstream group.
#     proxy_pass: dns_servers
#     # Can be used to add extra config blocks (multiline).
#     extra_parameters: |
#       proxy_responses 0;
```

## [Requirements](#requirements)

- pip packages listed in [requirements.txt](https://github.com/mullholland/ansible-role-nginx/blob/master/requirements.txt).


## [Context](#context)

This role is a part of many compatible roles. Have a look at [the documentation of these roles](https://mullholland.net) for further information.

Here is an overview of related roles:
![dependencies](https://raw.githubusercontent.com/mullholland/ansible-role-nginx/png/requirements.png "Dependencies")

## [Compatibility](#compatibility)

This role has been tested on these [container images](https://hub.docker.com/u/mullholland):

|container|tags|
|---------|----|
|[EL](https://hub.docker.com/repository/docker/mullholland/docker-centos-systemd/general)|all|
|[Amazon](https://hub.docker.com/repository/docker/mullholland/docker-amazonlinux-systemd/general)|Candidate|
|[Ubuntu](https://hub.docker.com/repository/docker/mullholland/docker-ubuntu-systemd/general)|all|
|[Debian](https://hub.docker.com/repository/docker/mullholland/docker-debian-systemd/general)|all|

The minimum version of Ansible required is 2.10, tests have been done to:

- The previous version.
- The current version.
- The development version.

If you find issues, please register them in [GitHub](https://github.com/mullholland/ansible-role-nginx/issues)

## [License](#license)

[MIT](https://github.com/mullholland/ansible-role-nginx/blob/master/LICENSE).

## [Author Information](#author-information)

[Mullholland](https://mullholland.net)

Please consider [sponsoring me](https://github.com/sponsors/mullholland).
