```apacheconf
upstream website_php {
    server fishapp.php.dev:9000;
}

server {
    listen 80;
    #listen 443 ssl;
    server_name fishapp.localhost;
    root /home/www/website/www/public;
    index index.php index.html index.htm;
    client_max_body_size 500M;
    sendfile off;
    #ssl_certificate /etc/nginx/ssl/server.crt;
    #ssl_certificate_key /etc/nginx/ssl/server.key;

    location / {
        # try to serve file directly, fallback to index.php
        try_files $uri /index.php$is_args$args;
    }

    location ~ ^/index\.php(/|$) {
        fastcgi_pass website_php;
        fastcgi_split_path_info ^(.+\.php)(/.*)$;
        include fastcgi_params;


        fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
        fastcgi_param DOCUMENT_ROOT $realpath_root;
        # Prevents URIs that include the front controller. This will 404:
        # http://domain.tld/index.php/some-path
        # Remove the internal directive to allow URIs like this
        internal;
        #fastcgi_param  HTTPS on;
    }

    # return 404 for all other php files not matching the front controller
    location ~ \.php$ {
        return 404;
    }

    access_log /dev/stdout vhosts;
    error_log  /dev/stderr;
}
```