# metabase-docker-nginx-ssl
steps to secure metabase on docker behind ssl nginx with let'sencrypt

## 1 set up metabase on docker

https://www.metabase.com/docs/latest/operations-guide/running-metabase-on-docker.html


## 2 Install nginx, certbot, get the certificate and set them up

first, install nginx

    sudo apt-get update
    sudo apt install nginx

allow http and https with your firewall, if enabled.

    sudo ufw allow 'Nginx Full'
 
 if you do not want the default nginx configuration, remove it
 
    sudo rm -rf /etc/nginx/sites-available/default
    sudo rm -rf /etc/nginx/sites-enabled/default

installl Let's encrypt certification bot, with the plugin for nginx verification:

    sudo add-apt-repository ppa:certbot/certbot  
    sudo apt update  
    sudo apt install python-certbot-nginx

obtain the certificate

    sudo certbot --nginx certonly

create the new conf

    sudo nano /etc/nginx/sites-available/metabase-site.conf

if your domain is `mydomain.com`, and the port where metabase is listening is 3000, and you want to redirect all traffic to https, write:

        server {
         listen [::]:80;
         listen 80;
    
         server_name mydomain.com;
    
         return 301 https://mydomain.com$request_uri;
     }
    
     server {
         listen [::]:443 ssl;
         listen 443 ssl;
    
         server_name mydomain.com;
    
         ssl_certificate /etc/letsencrypt/live/mydomain.com/fullchain.pem;
         ssl_certificate_key /etc/letsencrypt/live/mydomain.com/privkey.pem;
    
         location / {
             proxy_set_header X-Forwarded-Host $host;
             proxy_set_header X-Forwarded-Server $host;
             proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
             proxy_pass http://localhost:3000;
             client_max_body_size 100M;
         }
     }

enable this configuration :

    sudo ln -s /etc/nginx/sites-available/metabase-site.conf /etc/nginx/sites-enabled/metabase-site.conf

test that the configuration is OK:

    sudo nginx -t  
    
  restart nginx
  
    sudo service nginx restart
 or
 
    sudo systemctl reload nginx
    

test your certificate renewal procedure

    sudo certbot renew --dry-run

if this shows no error, certbot will renew the certificates automagically.


references:
1) https://www.cloudbooklet.com/install-metabase-on-ubuntu-18-04-with-nginx-and-ssl-google-cloud/
2) https://www.digitalocean.com/community/tutorials/how-to-secure-nginx-with-let-s-encrypt-on-ubuntu-18-04
3) https://www.digitalocean.com/community/tutorials/how-to-set-up-nginx-load-balancing-with-ssl-termination#:~:text=About%20SSL%20Termination,backend%20servers%20is%20in%20HTTP.
