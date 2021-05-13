# Nginx Learning Notes

This repository is composed of based on Nginx handbook of farhan.info.

"Sharing knowledge is the most fundamental act of friendship. Because it is a way you can give something without loosing something. — Richard Stallman"

Sample cases are created by Vagrant. Used vagrantfile can be fount at the repository.

***Nginx*** is high performance web server.

Main configuration file after installation is **`/etc/nginx/nginx.conf`**

## Basic Configuration: 

```
events{

}
http {
    server {
        listen 80;
        server_name sample-host.test
        return 200 "Hello there!\n";
    }
}
```

To validate configuration, you can use following command:

**`sudo nginx -t`**

To restart nginx server, you can use following command:

**`sudo systemctl restart nginx`**

## Multiple Server Configuration

```
http {
    server {
        listen 80;
        server_name a-domain.test;

        return 200 "Welcome to A area!\n";
    }


    server {
        listen 80;
        server_name b-domain.test;

        return 200 "Welcome to B area!\n";
    }
}
```

## To serve static content

```
events {
}
http {
    include /etc/nginx/mime.types;

    server {
        listen 80;
        server_name nginx-handbook.test;

        index index.html;
        root /srv/nginx-handbook-projects/static-demo;
    }

}
```

`root` director says location of static content.

`index` says entry point of the site.


`include /etc/nginx/mime.types;` is required to serve another mime type in the site. If you don't specify required mime types, these content won't be evaluated. 

`/etc/nginx/mime.types` is generic mime types.

## Location at Nginx

```
events {
}
http {
    server {
        listen 80;
        server_name nginx-handbook.test;
        location /agatha {
            return 200 "Miss Marple.\nHercule Poirot.\n";
        }
    }
}
```

The expression `location /agatha` accepts every text contains word `agatha` such as `agatha-christie`.

For exact match, sign `=` should be used like that.

```
 location = /agatha {
            return 200 "Miss Marple.\nHercule Poirot.\n";
        }
```

Regex match has more priority than a prefix match.


`set $<variable_name> <variable_value>;`

For example:

`set name "Farhan"`
`set age 25`

## Variables 

### Predefined

`return 200 "Host - $host\nURI - $uri\nArgs - $args\n";`

`curl http://nginx-handbook.test/user?name=Farhan`

`Host - nginx-handbook.test`
`URI - /user`
`Args - name=Farhan`

### User defined

```
set $name $arg_name; # $arg_<query string name>

return 200 "Name - $name\n";
```

`curl http://nginx-handbook.test?name=Farhan`

Output: Name - Farhan

## Redirection and Rewrites

### Redirection

```
location = /about_page {
    return 307 /index.html;
}
```

If you send a request to http://nginx-handbook.test/about_page, you'll be redirected to http://nginx-handbook.test/about.html:


### Rewrite

`rewrite /about_page /about.html;`

If you enter `http://nginx-handbook.test/about_page`, redirection won't be applied and content will be served at name of `/about_page`.

## Try for Multiple files

```
try_files /the-nginx-handbook.jpg /not_found;

    location /not_found {
        return 404 "sadly, you've hit a brick wall buddy!\n";
    }
```

If it doesn't exist, go to the /not_found location.

### Typical `try_files` usage

`try_files $uri $uri/ /not_found;`

## Logging

`/var/log/nginx/`

Reopen the log files:

`sudo nginx -s reopen`

### Allow the logging.
```
location = /admin {
        access_log /var/logs/nginx/admin.log;
            
        return 200 "this will be logged in a separate file.\n";
    }
```

### Disallow the logging.

```
location = /no_logging {
    access_log off;
            
        return 200 "this will not be logged.\n";
    }
```

## Reverse Proxy

```
events {
}
http {
    listen 80;
    server_name nginx-handbook.test

    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
    }
}
```

### Define User specification at config file.

```
user www-data;
events {

}
...
```

## Load balancer

```
events {
}
http {
    upstream backend_servers {
        server localhost:3001;
        server localhost:3002;
        server localhost:3003;
    }
    server {
        listen 80;
        server_name nginx-handbook.test;
        location / {
            proxy_pass http://backend_servers;
        }
    }
}
```

## How to optimize Nginx

### Workerprocess

```
worker_processes 2;
events {

}

...
```

A rule of thumb in determining the optimal number of worker processes is number of worker process = number of CPU cores.

### How to cache static-contents

```
 location ~* \.(css|js|jpg)$ {
        access_log off;
            
        add_header Cache-Control public;
        add_header Pragma public;
        add_header Vary Accept-Encoding;
        expires 1M;
}
```

## Deep Dive to Main Configuration File

```
user www-data;
worker_processes auto;
pid /run/nginx.pid;
include /etc/nginx/modules-enabled/*.conf;

events {
	worker_connections 768;
	# multi_accept on;
}
http {
...

access_log /var/log/nginx/access.log;
error_log /var/log/nginx/error.log;

...

include /etc/nginx/conf.d/*.conf;
include /etc/nginx/sites-enabled/*;

}
```

These two lines instruct NGINX to include any configuration files found inside the **/etc/nginx/conf.d/** and **/etc/nginx/sites-enabled/** directories.

There is another directory **/etc/nginx/sites-available/** that's meant to store configuration files for your virtual hosts. The **/etc/nginx/sites-enabled/** directory is meant for storing the symbolic links to the files from the **/etc/nginx/sites-available/** directory.

Sample:

-   `/etc/nginx/sites-available/`
    -   `site-a`
    -   `site-b`
    -   `site-c`

-   `/etc/nginx/sites-enabled/`
    -   `site-a -> /etc/nginx/sites-available/site-a`
    -   `site-c -> /etc/nginx/sites-available/site-c`


Sample creating soft link:
`ln -s /etc/nginx/sites-available/site-a /etc/nginx/sites-enabled/site-b`


Sample `site-a`:

```
server {
    listen 80;
    server_name nginx-handbook.test;

    root /srv/nginx-handbook-projects/static-demo;
}
```

Note:

Since main configuration has expression `include /etc/nginx/sites-enabled/*;` within `http` block, files which are under `sites-available` should start with `server`.




