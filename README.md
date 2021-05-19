# Set-up-Livestreaming-Server-RTMP
Setting up HLS live streaming server using NGINX + nginx-rtmp-module on Ubuntu!
This project is done on Ubuntu 18.04.

# The prerequisites:
Have a machine running ubuntu 18.04

## This guide will explain how to setup your own streaming server.
ssh to your machine using putty or powershell
#### Update and upgrade your machine:
```
 sudo apt-get update
 sudo apt-get upgrade
 ```
 #### Install the necessary tools to build nginx using the following command:
 ```
 sudo apt-get install build-essential libpcre3 libpcre3-dev libssl-dev zlib1g-dev
 ```
 ####   From your home directory, download the latest Nginx source code by this time the latest stable version of Nginx 1.20.0.
 You can find the latest version on the [nginx download page](http://nginx.org/en/download.html).
 ```
 wget http://nginx.org/download/nginx-1.20.0.tar.gz
 ls
 tar -zxvf nginx-1.20.0.tar.gz
 ```
 #### Next, get the RTMP module source code from git:
 ```
 wget https://github.com/sergey-dryabzhinsky/nginx-rtmp-module/archive/dev.zip
 ls
 sudo apt install unzip
 unzip dev.zip
 ```
 
 #### Now Compile/build nginx:
 ```
 ./configure --with-http_ssl_module --add-module=../nginx-rtmp-module-dev
 make
sudo make install
```
#### Create nginx configuration file
##### First I will backup the original nginx.conf file
```
sudo mv /usr/local/nginx/conf/nginx.conf /usr/local/nginx/conf/nginx.conf.original
```
#### Create and open nginx.conf file.  
```
sudo nano /usr/local/nginx/conf/nginx.conf
```
#### Add the following lines to it
```
worker_processes  auto;
events {
    worker_connections  1024;
}

# RTMP configuration
rtmp {
    server {
        listen 1935; # Listen on standard RTMP port
        chunk_size 4000;

        application show {
            live on;
            # Turn on HLS
            hls on;
            hls_path /nginx/hls/;
            hls_fragment 3;
            hls_playlist_length 60;
            # disable consuming the stream from nginx as rtmp
            deny play all;
        }
    }
}

http {
    sendfile off;
    tcp_nopush on;
    # aio on;
    directio 512;
    default_type application/octet-stream;

    server {
        listen 8080;

        location / {
            # Disable cache
            add_header 'Cache-Control' 'no-cache';

            # CORS setup
            add_header 'Access-Control-Allow-Origin' '*' always;
            add_header 'Access-Control-Expose-Headers' 'Content-Length';

            # allow CORS preflight requests
            if ($request_method = 'OPTIONS') {
                add_header 'Access-Control-Allow-Origin' '*';
                add_header 'Access-Control-Max-Age' 1728000;
                add_header 'Content-Type' 'text/plain charset=UTF-8';
                add_header 'Content-Length' 0;
                return 204;
            }

            types {
                application/dash+xml mpd;
                application/vnd.apple.mpegurl m3u8;
                video/mp2t ts;
            }

            root /nginx/;
        }
    }
}
```
#### Things you need to know about the nginx.conf file
1- Your application name is show (You can use any name)
2- The server listen to port 8080
3- The hls path is /nginx/hls



 

  
  
