# nginx-ec2-letsencrypt

https://pyliaorachel.github.io/blog/tech/system/2017/07/14/nginx-server-ssl-setup-on-aws-ec2-linux-with-letsencrypt.html

### get certbot
```
wget https://dl.eff.org/certbot-auto
chmod a+x certbot-auto
```

### conf => /etc/letsencrypt/configs/thetnaingaye.com.conf

```
# domains to retrieve certificate
domains = www.thetnaingaye.com

# increase key size
rsa-key-size = 2048

# the CA endpoint server
server = https://acme-v01.api.letsencrypt.org/directory

# the email to receive renewal reminders, IIRC
email = mmu1tn@gmail.com

# turn off the ncurses UI, we want this to be run as a cronjob
text = True
```

### install dependcies
```
sudo yum install python-pip python-dev
sudo pip install virtualenv
```

### run certbot
```
sudo service nginx stop
sudo ./certbot-auto --standalone --no-bootstrap --config /etc/letsencrypt/configs/thetnaingaye.com.conf certonly
```


### edit nginx.conf
```
# For more information on configuration, see:
#   * Official English Documentation: http://nginx.org/en/docs/
#   * Official Russian Documentation: http://nginx.org/ru/docs/

user nginx;
worker_processes auto;
error_log /var/log/nginx/error.log;
pid /run/nginx.pid;

# Load dynamic modules. See /usr/share/nginx/README.dynamic.
include /usr/share/nginx/modules/*.conf;

events {
    worker_connections 1024;
}

http {
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile            on;
    tcp_nopush          on;
    tcp_nodelay         on;
    keepalive_timeout   65;
    types_hash_max_size 2048;

    include             /etc/nginx/mime.types;
    default_type        application/octet-stream;

    # Load modular configuration files from the /etc/nginx/conf.d directory.
    # See http://nginx.org/en/docs/ngx_core_module.html#include
    # for more information.
    include /etc/nginx/conf.d/*.conf;

    server {
        listen       80 default_server;
        listen       [::]:80 default_server;
        server_name  _;
        root         /usr/share/nginx/html;

        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;

        location / {
        }

        error_page 404 /404.html;
            location = /40x.html {
        }

        error_page 500 502 503 504 /50x.html;
            location = /50x.html {
        }
    }
	server {
	 	server_name 3.86.116.30;
	 	return 301 https://www.thetnaingaye.com;
	}
# Settings for a TLS enabled server.

    server {
        listen       443 ssl http2 default_server;
        listen       [::]:443 ssl http2 default_server;
        server_name  _;
        root         /usr/share/nginx/html;

        ssl_certificate "/etc/letsencrypt/live/www.thetnaingaye.com/fullchain.pem";
        ssl_certificate_key "/etc/letsencrypt/live/www.thetnaingaye.com/privkey.pem";
        ssl_session_cache shared:SSL:1m;
        ssl_session_timeout  10m;
        ssl_ciphers HIGH:!aNULL:!MD5;
        ssl_prefer_server_ciphers on;

        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;

        location / {
        }

        error_page 404 /404.html;
            location = /40x.html {
        }

        error_page 500 502 503 504 /50x.html;
            location = /50x.html {
        }

#	return 301 $scheme://www.thetnaingaye.com$request_uri;
    }

}
```

### renew
```
sudo ./certbot-auto renew --pre-hook "service nginx stop" --post-hook "service nginx start"
```
