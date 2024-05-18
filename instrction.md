 <!-- for avoid error you can copy and open with Notepad -->

1.STEPS

sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# update the packages
sudo apt-get update

# Install Docker Packages
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

2.STEPS  

1. Configure Docker Registry
```sh
mkdir ~/docker-registry
cd ~/docker-registry
mkdir data

# Create docker compose config file
vim docker-compose.yml
```
3.STEPS

Configration for `docker-compose.yml` 

```yml
version: '3'
services:
  registry:
    image: registry:2
    ports:
    - "5000:5000"
    environment:
      REGISTRY_STORAGE_FILESYSTEM_ROOTDIRECTORY: /data
    volumes:
      - ./data:/data
```


4.STEPS

1. Nginx and SSL Setup
```sh
sudo apt install nginx
sudo vim etc/nginx/sites-available/<your_site>
sudo vim etc/nginx/sites-available/hub.omkarjagtap.work

```



5.STEPS

Configration for nginx

```
server_name your_domain.com

location / {
    proxy_pass                          http://localhost:5000;
    proxy_set_header  Host              $http_host;   # required for docker client's sake
    proxy_set_header  X-Real-IP         $remote_addr; # pass on real client's IP
    proxy_set_header  X-Forwarded-For   $proxy_add_x_forwarded_for;
    proxy_set_header  X-Forwarded-Proto $scheme;
    proxy_read_timeout                  900;
}
```


6.STEPS


Test & Restart Nginx

```
# Check NGINX config
sudo nginx -t

# Restart NGINX
sudo nginx -s reload
```



7.STEPS

1. SSL Certificate
```
sudo add-apt-repository ppa:certbot/certbot
sudo apt-get update
sudo apt-get install python3-certbot-nginx
sudo certbot --nginx
```
1. Run the docker service
```
sudo docker compose up
```
visit `https://yout_domain.com/v2` 

1. Setting up Authentication
```sh
sudo apt install apache2-utils -y
mkdir ~/docker-registry/auth
cd ~/docker-registry/auth
```
`htpasswd -Bc registry.password <username>` 
htpasswd -Bc registry.password omkar

```sh
vim ~/docker-registry/docker-compose.yml
```
```diff
version: '3'

services:
  registry:
    image: registry:2
    ports:
    - "5000:5000"
    environment:
+      REGISTRY_AUTH: htpasswd
+      REGISTRY_AUTH_HTPASSWD_REALM: Registry
+      REGISTRY_AUTH_HTPASSWD_PATH: /auth/registry.password
       REGISTRY_STORAGE_FILESYSTEM_ROOTDIRECTORY: /data
    volumes:
+      - ./auth:/auth
       - ./data:/data
```
```sh
docker-compose up
```
Increase File Upload Limit Size for Nginx

```sh
vim /etc/nginx/nginx.conf
```
```diff
http {
+ client_max_body_size 16384m;
}
```
```
sudo nginx -s reload
```
### Publishing images from our local machine
```sh
docker login your_domain
```
```sh
docker tag image <your_domain>/<repo>
docker push <your_domain>/<repo>
```


used that images
docker pull hub.omkarjagtap.worl/my-redis


login on the git bash

echo "admin" | docker login hub.omkarjagtap.work --username omkar --password-stdin


how things work well in this scenario working
1. on local machine -inside a git bash - the redis pull from the  dockerhub.registry

2. docker tag   hub.omkarjagtap.worl/my-redis


3.docker push   hub.omkarjagtap.worl/my-redis







