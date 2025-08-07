# NGINX notes

> free , open source , BSD license , developed by Igor Sysoev to solve _C10K_ problem "challenge to handel 10000 concurent connection" _concurent connection is multiple connections at the same time_ 

> used as 

>> 1. webserver : is a system (hardware or software or combination) that delivers web content (media,httml,css,php,python,ruby) to the user over the internet. 

            _hardware ==> computer store the web server software and website files_

            _software* ==> http or https server which handels the delivery of files_
 
            _static website ==> the elements "html,css,javaS,fixed media" on the page remain the same_
 
            _dynamic website ==> the elements "python,php,ruby,node.js" are generated in real time or changed based on the user_
 
            _static uses client side languages "html,JS,css" and dynamic uses server side languages "php,ruby,python"_
 
            _https encrypted TLS/SSL "transport layer security and secure sockets layer" "protocols used to encrypt communication betwwen client and server"_
            
>>>> #### http response status code            
  
  
>>> ### NGINX Worker Processes
     Worker processes handling client connections and processing requests. _The master process manages these workers_, including starting, stopping, and reloading them.

>>> ### The Event Loop
     Core of Worker Efficiency: At the heart of each worker process is an event loop. This loop continuously monitors for I/O events (e.g., new connections, data ready to be read, data ready to be written) on all active connections. 

>>> ### Workflow:
    1. The worker process enters the event loop.
    2. It listens for I/O events on active connections.
    3. When an event (e.g., a new client connection, data received from a client, data sent to a client) is detected, the worker processes the relevant action for that specific event.
    4. After processing, the worker returns to the event loop to await the next event on any of its managed connections.


>> 2. load balancer : distribute incoming requests across multiple servers , nginx prevent backend server from becoming a bottleneck.
```
       upstream backend {
            server backend1.example.com;
            server backend2.example.com max_fails=3 fail_timeout=30s;
            }


        server {
            listen 80;
            location / {
                proxy_pass http://backend;
            }
        }
```

>> 3.reverse proxy : accepts client requests, applies routing or SSL offloading, then forwards them to one or more backend servers.
```
        server {
            listen 443 ssl;
            server_name example.com;


            ssl_certificate     /etc/nginx/ssl/example.crt;
            ssl_certificate_key /etc/nginx/ssl/example.key;


            location / {
                proxy_pass       http://internal_app;
                proxy_set_header Host $host;
                proxy_set_header X-Real-IP $remote_addr;
        }
    }
```

>> 4. forward proxy : sits between clients and the internet, filtering or anonymizing outbound requests. Configure Nginx to restrict sites or mask client IPs for privacy.
```
        server {
                listen 3128;


            resolver 8.8.8.8;
            proxy_pass_request_headers on;


            location / {
                proxy_pass $scheme://$http_host$request_uri;
                proxy_hide_header Proxy-Authorization;
            }
        }
```

>> 5. caching : reduces response times and eases load on backend services. Define a cache zone, set key parameters, and control how responses are stored.
```
        proxy_cache_path /var/cache/nginx levels=1:2 keys_zone=my_cache:10m 
                    inactive=60m max_size=1g;


        server {
            listen 80;
            location / {
                proxy_cache my_cache;
                proxy_pass http://backend;
                proxy_cache_valid 200 302 10m;
                proxy_cache_valid 404 1m;
            }
        }
```


> nginx configuration 
* /etc/nginx/nginx.conf

>> nginx.conf Structure
1. Global settings 
Define user permissions, worker processes, PID file location, compression, caching, and more.
```
user www-data; # system user is www-data
worker_processes auto; # number of worker processes is _auto_ matches CPU cores
pid /run/nginx.pid; # pathe to the master process id 
```

2. events block
Controls Nginx’s event model and the maximum number of simultaneous connections per worker.
```
events {
    worker_connections 1024; # max connections worker can handel 
}
```

3. http block
Contains HTTP directives for logging, timeouts, compression, MIME types, and includes for server blocks.
```
http {
    sendfile        on;
    tcp_nopush      on;
    tcp_nodelay     on;
    keepalive_timeout 65;
    types_hash_max_size 2048;
    gzip            on;


    log_format main '$remote_addr - $remote_user [$time_local] '
                    '"$request" $status $body_bytes_sent '
                    '"$http_referer" "$http_user_agent"';


    include /etc/nginx/mime.types;
    default_type application/octet-stream;


    # Include virtual host definitions
    include /etc/nginx/conf.d/*.conf;
    include /etc/nginx/sites-enabled/*;
}
```
4. server block
Configures how Nginx responds to requests for specific domain names or IP addresses (virtual hosts).
```
server {
    listen 80; # port of incoming traffic 443 for https 80 for http
    server_name example.com www.example.com; # domain name

    root /var/www/example.com/html; # path where nginx will access files 
    index index.html; # default index file served when directory is requested

    location / {
        try_files $uri $uri/ =404;
    }
}
```
#### Explanation of the try_files directive:

`$uri`:
* Nginx first attempts to find a file that exactly matches the requested URI relative to the root directive defined for the server or this location block. For example, if a request is for /image.jpg and the root is /var/www/html, Nginx will look for /var/www/html/image.jpg.

`$uri/`:
* If the $uri is not found, Nginx then attempts to treat the requested URI as a directory and looks for an index file within that directory. For example, if a request is for /docs/ and the index file is index.html, Nginx will look for /var/www/html/docs/index.html.

`=404`:
* If neither a matching file nor a directory with an index file is found, Nginx will return a 404 Not Found error. This is the fallback action if all preceding try_files attempts fail.


### nginx directory structure
| File/Directory | Description |
|---|---|
| `/etc/nginx/nginx.conf` | Main configuration file |
| `/etc/nginx/sites-available/` | Stores individual server block files |
| `/etc/nginx/sites-enabled/` | Symbolic links to enabled sites from sites-available |
| `/etc/nginx/conf.d/` | Additional configuration snippets (e.g., SSL, load balancing) |
| `/var/www/...` | Default web content roots (Debian/Ubuntu) |
| `/usr/share/nginx/html` | Default web root (RHEL/CentOS) |
| `/etc/nginx/mime.types` | MIME type definitions |
| `/run/nginx.pid` | PID file location |
| `/var/log/nginx/` | Access and error logs |


### using firewall 
> Debian

>> [ufw](https://help.ubuntu.com/community/UFW)

>> ufw enable
>> ufw allow 80/tcp
>> ufw reload 
>> ufw status numbered
>> ufw delete <rule_number>
>> ufw status

> Redhat

>> [firewalld](https://firewalld.org/documentation/)

>> systemctl start firewall-cmd
>> systemctl enable firewall-cmd
>> firewall-cmd --permanent --add-port=80/tcp
>> firewall-cmd --reload 
>> firewall-cmd --permanent --remove-port=80/tcp
>> firewall-cmd --list-all

### Redirect and Rewrite
> Redirect (return) issues an HTTP status code (e.g., 301) back to the client and changes what appears in their browser’s address bar.

>> forward every request from one domain to another
```
server {
    listen       80;
    server_name  honda.cars.com;


    return 301 https://cars.honda.com$request_uri;


    root  /var/www/example.com/html;
    index index.html;


    location / {
        try_files $uri $uri/ =404;
    }
}
```
>> HTTP → HTTPS Redirect
```
server {
    listen       80;
    server_name  honda.cars.com;


    return 301 https://$host$request_uri;
}


server {
    listen       443 ssl;
    server_name  honda.cars.com;


    ssl_certificate     /etc/ssl/certs/honda.cars.com.pem;
    ssl_certificate_key /etc/ssl/certs/honda.cars.com-key.pem;


    root  /var/www/;
}
```
>> Redirect Single-Page
```
server {
    listen       80;
    server_name  honda.cars.com;


    root  /var/www/example.com/html;
    index index.html;


    location /civic-type-r {
        return 301 https://cars.honda.com$request_uri;
    }
}
```

> Rewrite silently alters the URI before Nginx processes it, keeping the user’s URL intact.
*modify incoming URLs before Nginx searches for files or forwards to upstream servers—ideal for cleanup rules or legacy support.*


>> convert long URIs into friendly
```
server {
    listen       80;
    server_name  honda.cars.com;


    root  /var/www/example.com/html;
    index index.html;


    rewrite ^/sports-car-civic-type-r$ /type-r permanent;
}
```

>> Rewriting Directory Paths
```
server {
    listen       80;
    server_name  honda.cars.com;


    root  /var/www/example.com/html;
    index index.html;


    location /pics {
        rewrite ^/pics/(.*)$ /images/$1 permanent;
    }
}
```

