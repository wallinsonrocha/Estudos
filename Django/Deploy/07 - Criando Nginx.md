# Deploy

## http
Sem o SSL. Basta seguir o tutorial abaixo.
```
# https://www.nginx.com/blog/using-free-ssltls-certificates-from-lets-encrypt-with-nginx/
#
# REPLACES
# ____REPLACE_ME_WITH_YOUR_OWN_DOMAIN____ = Replace with your domain
# __PROJECT_FOLDER__ = Replace with the path to the folder for the project
# __STATIC_FOLDER_PATH__ = Replace with the path to the folder for static files
# __MEDIA_FOLDER_PATH__ = Replace with the path to the folder for media files
# __SOCKET_NAME__ = Replace with your unix socket name
# 
# Set timezone
# List - timedatectl list-timezones
# sudo timedatectl set-timezone America/Sao_Paulo
#
# HTTP
server {
  listen 80;
  listen [::]:80;
  server_name ____REPLACE_ME_WITH_YOUR_OWN_DOMAIN____;

  # Add index.php to the list if you are using PHP
  index index.html index.htm index.nginx-debian.html index.php;
  
  # ATTENTION: __STATIC_FOLDER_PATH__
  location /static {
    autoindex on;
    alias __STATIC_FOLDER_PATH__;
  }

  # ATTENTION: __MEDIA_FOLDER_PATH__ 
  location /media {
    autoindex on;
    alias __MEDIA_FOLDER_PATH__;
  }

  # ATTENTION: __SOCKET_NAME__
  location / {
    proxy_pass http://unix:/run/__SOCKET_NAME__;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection 'upgrade';
    proxy_set_header Host $host;
    proxy_cache_bypass $http_upgrade;
  }

  # deny access to .htaccess files, if Apache's document root
  # concurs with nginx's one
  #
  location ~ /\.ht {
    deny all;
  }

  location ~ /\. {
    access_log off;
    log_not_found off;
    deny all;
  }

  gzip on;
  gzip_disable "msie6";

  gzip_comp_level 6;
  gzip_min_length 1100;
  gzip_buffers 4 32k;
  gzip_proxied any;
  gzip_types
    text/plain
    text/css
    text/js
    text/xml
    text/javascript
    application/javascript
    application/x-javascript
    application/json
    application/xml
    application/rss+xml
    image/svg+xml;

  access_log off;
  #access_log  /var/log/nginx/____REPLACE_ME_WITH_YOUR_OWN_DOMAIN____-access.log;
  error_log   /var/log/nginx/____REPLACE_ME_WITH_YOUR_OWN_DOMAIN____-error.log;
}
```

## https
```
# https://www.nginx.com/blog/using-free-ssltls-certificates-from-lets-encrypt-with-nginx/
#
# REPLACES
# ____REPLACE_ME_WITH_YOUR_OWN_DOMAIN____ = Replace with your domain
# __PROJECT_FOLDER__ = Replace with the path to the folder for the project
# __STATIC_FOLDER_PATH__ = Replace with the path to the folder for static files
# __MEDIA_FOLDER_PATH__ = Replace with the path to the folder for media files
# __SOCKET_NAME__ = Replace with your unix socket name
# 
# For letsencrypt and Ubuntu:
# sudo apt update -y && sudo apt upgrade -y && sudo apt autoremove -y
# sudo apt install nginx certbot python3-certbot-nginx -y
# sudo openssl dhparam -out /etc/ssl/certs/dhparam.pem 4096
# sudo systemctl stop nginx
# sudo certbot certonly --standalone -d ____REPLACE_ME_WITH_YOUR_OWN_DOMAIN____
# sudo chmod -R 755 __PROJECT_FOLDER__
# sudo nano /etc/nginx/sites-available/____REPLACE_ME_WITH_YOUR_OWN_DOMAIN____
# Add contents of this file
# sudo ln -s /etc/nginx/sites-available/____REPLACE_ME_WITH_YOUR_OWN_DOMAIN____ /etc/nginx/sites-enabled/____REPLACE_ME_WITH_YOUR_OWN_DOMAIN____
#
# Set timezone
# List - timedatectl list-timezones
# sudo timedatectl set-timezone America/Sao_Paulo
#
# HTTP
server {
  listen 80;
  listen [::]:80;
  server_name ____REPLACE_ME_WITH_YOUR_OWN_DOMAIN____;
  return 301 https://$host$request_uri;
}

# HTTPS
server {
  listen 443 ssl http2;
  listen [::]:443 ssl http2;

  server_name ____REPLACE_ME_WITH_YOUR_OWN_DOMAIN____;

  ssl_certificate /etc/letsencrypt/live/____REPLACE_ME_WITH_YOUR_OWN_DOMAIN____/fullchain.pem;
  ssl_certificate_key /etc/letsencrypt/live/____REPLACE_ME_WITH_YOUR_OWN_DOMAIN____/privkey.pem;
  ssl_trusted_certificate /etc/letsencrypt/live/____REPLACE_ME_WITH_YOUR_OWN_DOMAIN____/chain.pem;

  # Improve HTTPS performance with session resumption
  ssl_session_cache shared:SSL:10m;
  ssl_session_timeout 5m;

  # Enable server-side protection against BEAST attacks
  ssl_prefer_server_ciphers on;
  ssl_ciphers ECDH+AESGCM:ECDH+AES256:ECDH+AES128:DH+3DES:!ADH:!AECDH:!MD5;

  # Disable SSLv3
  ssl_protocols TLSv1 TLSv1.1 TLSv1.2;

  # Diffie-Hellman parameter for DHE ciphersuites
  # $ sudo openssl dhparam -out /etc/ssl/certs/dhparam.pem 4096
  ssl_dhparam /etc/ssl/certs/dhparam.pem;

  # Enable HSTS (https://developer.mozilla.org/en-US/docs/Security/HTTP_Strict_Transport_Security)
  add_header Strict-Transport-Security "max-age=63072000; includeSubdomains";

  # Enable OCSP stapling (http://blog.mozilla.org/security/2013/07/29/ocsp-stapling-in-firefox)
  ssl_stapling on;
  ssl_stapling_verify on;
  resolver 8.8.8.8 8.8.4.4 valid=300s;
  resolver_timeout 5s;

  # Add index.php to the list if you are using PHP
  index index.html index.htm index.nginx-debian.html index.php;
  
  # ATTENTION: __STATIC_FOLDER_PATH__
  location /static {
    autoindex on;
    alias __STATIC_FOLDER_PATH__;
  }

  # ATTENTION: __MEDIA_FOLDER_PATH__ 
  location /media {
    autoindex on;
    alias __MEDIA_FOLDER_PATH__;
  }

  # ATTENTION: __SOCKET_NAME__
  location / {
    proxy_pass http://unix:/run/__SOCKET_NAME__;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection 'upgrade';
    proxy_set_header Host $host;
    proxy_cache_bypass $http_upgrade;
  }

  # deny access to .htaccess files, if Apache's document root
  # concurs with nginx's one
  #
  location ~ /\.ht {
    deny all;
  }

  location ~ /\. {
    access_log off;
    log_not_found off;
    deny all;
  }

  gzip on;
  gzip_disable "msie6";

  gzip_comp_level 6;
  gzip_min_length 1100;
  gzip_buffers 4 32k;
  gzip_proxied any;
  gzip_types
    text/plain
    text/css
    text/js
    text/xml
    text/javascript
    application/javascript
    application/x-javascript
    application/json
    application/xml
    application/rss+xml
    image/svg+xml;

  access_log off;
  #access_log  /var/log/nginx/____REPLACE_ME_WITH_YOUR_OWN_DOMAIN____-access.log;
  error_log   /var/log/nginx/____REPLACE_ME_WITH_YOUR_OWN_DOMAIN____-error.log;
}
```

## Criando arquivo no servidor
```
sudo nano /etc/nginx/sites-available/<nome>
```
Após isso, basta copiar os arquivos nesse novo arquivo criado.

## Ativando link do projeto
Após copiar, basta devemos tornar o site reconhecido pelo Nginx.

Removendo a pasta default do Nginx.
```
sudo rm /etc/nginx/sites-enabled/default
```

Colocando o arquivo para ser ativado.
```
sudo ln -s /etc/nginx/sites-available/<nome> /etc/nginx/sites-enabled/<nome>
```

Para finalizar
```
sudo nginx -t
sudo systemctl restart nginx
```