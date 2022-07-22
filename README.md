## Installation

Installation follows roughly this tutorial:

https://www.digitalocean.com/community/tutorials/how-to-set-up-a-video-streaming-server-using-nginx-rtmp-on-ubuntu-20-04

First, create a VPS with Ubuntu 20

```
sudo apt update
sudo apt install -y nginx
sudo apt install -y libnginx-mod-rtmp
```

etc/nginx/nginx.conf:

```nginx
user www-data;
worker_processes auto;
pid /run/nginx.pid;
include /etc/nginx/modules-enabled/*.conf;

events {
	worker_connections 768;
	# multi_accept on;
}

http {

	##
	# Basic Settings
	##

	sendfile on;
	tcp_nopush on;
	tcp_nodelay on;
	keepalive_timeout 65;
	types_hash_max_size 2048;
	# server_tokens off;

	# server_names_hash_bucket_size 64;
	# server_name_in_redirect off;

	include /etc/nginx/mime.types;
	default_type application/octet-stream;

	##
	# SSL Settings
	##

	ssl_protocols TLSv1 TLSv1.1 TLSv1.2 TLSv1.3; # Dropping SSLv3, ref: POODLE
	ssl_prefer_server_ciphers on;

	##
	# Logging Settings
	##

	access_log /var/log/nginx/access.log;
	error_log /var/log/nginx/error.log;

	##
	# Gzip Settings
	##

	gzip on;

	# gzip_vary on;
	# gzip_proxied any;
	# gzip_comp_level 6;
	# gzip_buffers 16 8k;
	# gzip_http_version 1.1;
	# gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;

	##
	# Virtual Host Configs
	##

	include /etc/nginx/conf.d/*.conf;
	include /etc/nginx/sites-enabled/*;
}

rtmp {
        server {
                listen 1935;
                chunk_size 4096;
                allow publish all;
				        allow play all;
				
                application live {
                        live on;
						            hls on;
                        hls_path /var/www/html/stream/hls;
                        hls_fragment 3;
                        hls_playlist_length 60;
						            record all;  
            			      record_path /tmp/record/;  
            			      record_suffix ___%y_%m_%d__%H_%M_%S.flv;  
            			      exec_record_done sudo /tmp/record/record.sh $path $basename;
                }
        }
}
```

### Recording

```
mkdir /tmp/record
touch /tmp/record.sh
chmod +x /tmp/record.sh
```

Paste the following to `/tmp/record.sh`:

```sh
 #!/bin/bash 
ffmpeg -i $1 -c copy /tmp/record/$2.mp4;
duration=`ffprobe -v error -show_entries format=duration -of default=noprint_wrappers=1:nokey=1 /tmp/record/$2.mp4`
cp /tmp/record/$2.mp4 /media/elektron/videos/$2__$duration.mp4;
rm /tmp/record/$2.flv
rm /tmp/record/$2.mp4
```

