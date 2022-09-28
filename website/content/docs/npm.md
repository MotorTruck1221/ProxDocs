---
title: "Nginx Proxy Manager"
date: 2022-08-31T18:28:11Z
draft: false
---
[Nginx Proxy Manager](https://nginxproxymanager.com/) (NPM). This is a docker container that will allow us to easily manage our Nginx instances. We will be using this to add SSL to our site as well as to proxy our site to our domain.

Assuming you are running a Linux system based off of Debian, run the following command to install some prerequisites:
```sh
$ sudo apt install -y apt-transport-https ca-certificates curl software-properties-common nano curl wget
```
Then, add the GPG keys
```sh
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```
After which add the Docker repository
```sh
$ echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```
Then, update your package list
```sh
$ sudo apt update
```
After all of this is done, we are going to make sure you are goin to install Docker from the repository added. And not just the base Ubuntu docker package
```sh
$ apt-cache policy docker-ce
```
After all of this we can FINALLY install docker and docker compose (which we will need later)
```sh
$ sudo apt install docker-ce docker-compose-plugin
```
<span style="color:red">These next steps are OPTIONAL</span>:<br>
We are going to add the current user to the docker group so we don't have to use sudo every time we want to run a docker command. This is not recommended for production environments, but for development it is fine.
```sh
$ sudo usermod -aG docker ${USER}
```
Then, we are going to log out and log back in so the group change takes effect.
```sh
$ su - ${USER}
```
<span style="color:red">End Optional Config</span><br>
Now we are going to install [Nginx Proxy Manager](https://nginxproxymanager.com/) (NPM).
- First, we are going to create a directory for NPM to store its data in
```sh
$ mkdir -p ~/npm/ && mkdir -p ~/npm/data && mkdir -p ~/npm/letsencrypt && cd ~/npm/
```
- Second we are going to create a docker-compose.yml file
```sh
touch docker-compose.yml
```
- Third we are going to add the following to the docker-compose.yml file <span style="color:red">NOTE: Do NOT CHANGE THE PORT NUMBERS THI WILL GO VERY POORLY</span><br>
```sh 
$ nano docker-compose.yml
```
Then copy this config into the file
```yml
version: "3"
services:
  app:
    image: 'jc21/nginx-proxy-manager:latest'
    restart: unless-stopped
    ports:
      # These ports are in format <host-port>:<container-port>
      - '80:80' # Public HTTP Port
      - '443:443' # Public HTTPS Port
      - '81:81' # Admin Web Port
      # Add any other Stream port you want to expose
      # - '21:21' # FTP

    # Uncomment the next line if you uncomment anything in the section
    # environment:
      # Uncomment this if you want to change the location of 
      # the SQLite DB file within the container
      # DB_SQLITE_FILE: "/data/database.sqlite"

      # Uncomment this if IPv6 is not enabled on your host
      # DISABLE_IPV6: 'true'

    volumes:
      - ./data:/data
      - ./letsencrypt:/etc/letsencrypt
```
or alternatively you can use the following command to create the file
```sh
$ curl https://hastebin.com/raw/ivulifoxer > docker-compose.yml
```
If you wish to use a different database beside sqlite3 you can find the instructions [here](https://nginxproxymanager.com/setup/#using-mysql-mariadb-database)
- Fourth we are now going to start NPM
```sh
$ sudo docker compose up -d
```
- Finally we are going to open up the admin page in our browser at: `http://(your server ip here):81` removing the parentheses and adding you server IP. <br>
 Upon opening the admin panel you will be prompted for an email and password the default email address is: <br> `admin@example.com` <br> and the default password is: <br>
 `changeme` <br>
 You will be prompted to change the password and email address. <br>
 After this setup we can get to adding your proxy's to NPM.
 - First we are going to setup ssl. I am really lazy so here is a video that is going to show you how to do this(not mine): [Here](https://youtu.be/rj7DZdWMK2k)
  - Second we are going to add a proxy proxy host to NPM.
  Click on Proxy Hosts in the left hand menu and then click on the + button in the top right corner. <br>
  ![Proxy Hosts](/TNproxhosts.png)
  ![Proxy Hosts](/TNaddproxhosts.png)
  Then fill out the form as follows: <br>
  Your domain or subdomain: <br>
  ![Proxy Hosts form](/TNproxform.png)
  Leave the http field alone (unless you for some reason have https on)
  Fill out the Foward Hostname/IP field with the IP of where your proxy is hosted. <br>
  For foward port change it to the port your proxy is on. <br>
  Turn on Block Common Explotis switch. <br>
  Then Turn on the Websockets Support switch. <br>
  Now click on the SSL tab and select the SSL certificate you created earlier(from the video). <br>
  After which Click on the advanced tab. And fill out the Custom Nginx configuration field with the following: <br>
```nginx
  location / { 
                # Upgrade WebSockets
                proxy_set_header Upgrade $http_upgrade;
                proxy_set_header Connection 'Upgrade';
                # Increase header buffer
                proxy_connect_timeout 10; 
                proxy_send_timeout 90; 
                proxy_read_timeout 90; 
                proxy_buffer_size 128k;
                proxy_buffers 4 256k;
                proxy_busy_buffers_size 256k;
                proxy_temp_file_write_size 256k;
                proxy_pass http://sameipaswhatyourproxyipis:sameportaswhatyourproxyportis; # change this to the port UltraViolet is listening on

            # The small block below will block googlebot
            if ($http_user_agent ~ (Googlebot)) {
                return 403;
            }
        }
```
After this you can click on the save button and you are done. <br>

## Authors
- [MotorTruck1221](https://github.com/motortruck1221)
