## Creating Motion Server

Use PI Imager to install latest full verion of PI-OS


Building a deb package
A script has also been written to allow users to create their own deb package from either a tagged release or the most current master branch of the code. Download the script by using the following command. wget https://raw.githubusercontent.com/Motion-Project/motion-packaging/master/build.sh
Review the script and specify the following few optional parameters. Username, EmailAddress, branch, install, arch. If the parameters are not provided, the script will create defaults.
Once the script has been run and the deb file is created, install it as described above.




### Add required Motion libraries

```
sudo apt install autoconf automake autopoint build-essential pkgconf libtool libzip-dev libjpeg-dev git libavformat-dev libavcodec-dev libavutil-dev libswscale-dev libavdevice-dev libopencv-dev libwebp-dev gettext libmicrohttpd-dev libmariadb-dev libcamera-dev libcamera-tools libcamera-v4l2 libasound2-dev libpulse-dev libfftw3-dev mariadb-client mariadb-client-core mariadb-common mariadb-server -y
```
```
cd ~
git clone https://github.com/Motion-Project/motion.git
cd motion
autoreconf -fiv
./configure
make
sudo make install
```
### Installed in
 /usr/bin/mkdir -p '/usr/local/var/lib/motion'
 /usr/bin/install -c -m 644 data/motion-dist.conf data/camera1-dist.conf data/camera2-dist.conf data/camera3-dist.conf data/sound1-dist.conf '/usr/local/var/lib/motion'
 /usr/bin/mkdir -p '/usr/local/var/lib/motion/webcontrol'
 /usr/bin/install -c -m 644 data/webcontrol/samplepage.html '/usr/local/var/lib/motion/webcontrol'


Add supporting STIG/SCAP required libraries
```
sudo apt install opensc libpam-pkcs11 auditd ufw libpam-pwquality apparmor vlock aide chrony -y
```
NGINX Install
```
sudo apt install nginx-common nginx-core libnginx-mod-rtmp -y
```
Append to /etc/nginx/nginx.conf
```
sudo nano /etc/nginx/nginx.conf
```
```
user www-data;
worker_processes auto;
worker_rlimit_nofile 65535;
pid /run/nginx.pid;
include /etc/nginx/modules-enabled/*.conf;
 
events {
    worker_connections 1024;
    multi_accept on;
    use epoll;
}
 
http {
    sendfile on;
    tcp_nopush on;
    types_hash_max_size 2048;
    include /etc/nginx/mime.types;
    default_type application/octet-stream;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2 TLSv1.3;
    ssl_prefer_server_ciphers on;
    access_log /var/log/nginx/access.log;
    error_log /var/log/nginx/error.log;
    gzip on;
    gzip_types text/plain application/json application/javascript text/css application/xml text/javascript;
    include /etc/nginx/conf.d/*.conf;
    include /etc/nginx/sites-enabled/*;
 
    server {
        listen 9090;
 
        location / {
            root /var/www/html;
            index index.html;
        }
 
        # HLS Streams für pi
        location /hlspi/ {
            root /var/hls;
            types {
                application/vnd.apple.mpegurl m3u8;
                video/mp2t ts;
            }
            add_header Cache-Control no-cache;
        }
 
        # HLS Streams für zero
        location /hlszero/ {
            root /var/hls;
            types {
                application/vnd.apple.mpegurl m3u8;
                video/mp2t ts;
            }
            add_header Cache-Control no-cache;
        }
 
        access_log /var/log/nginx/access.log;
        error_log /var/log/nginx/error.log;
    }
}
 
rtmp {
    server {
        listen 1935;
        timeout 30s;
        chunk_size 4096;
        allow publish all;
        allow play all;
        play_restart on;
        idle_streams off;
        drop_idle_publisher 60s;
        max_message 8M;
        max_connections 512;
        respawn on;
 
        application pi {
            live on;
            record off;
 
            hls on;
            hls_path /var/hls/hlspi;
            hls_fragment 8;
            hls_playlist_length 16;
            hls_type live;
            sync 100ms;
            hls_cleanup on;
        }
 
        application zero {
            live on;
            record off;
 
            hls on;
            hls_path /var/hls/hlszero;
            hls_fragment 8;
            hls_playlist_length 16;
            hls_type live;
            sync 100ms;
            hls_cleanup on;
 
        }
    }
}
```
####  Testing step  
libcamera-hello


## Configuration
Default Dir /usr/local/var/lib/motion

```
sudo mkdir /usr/local/var/lib/motion/conf.d
```

```
sudo cp /usr/local/var/lib/motion/camera1-dist.conf /usr/local/var/lib/motion/conf.d/camera.conf
```
```
sudo nano /usr/local/var/lib/motion/conf.d/camera.conf
```
####  Create Motion Service
```
sudo nano /etc/systemd/system/motion.service
```
```
[Unit]
Description=motion v5 Server
After=network.target
Wants=network-online.target
StartLimitIntervalSec=3
StartLimitBurst=5

[Service]
Type=simple
ExecStartPre=/bin/sleep 1
Nice=-12
Restart=always
RestartSec=5
User=root
ExecStart=/usr/local/bin/motion

[Install]
WantedBy=multi-user.target
```
```
sudo chmod 644 /etc/systemd/system/motion.service
```

/usr/local/bin/motion

Finally, tell systemd to run it at boot:
```
sudo systemctl enable motion.service
```

PiZero with Motion locally installed and streamed into master motion server
```
netcam_url http://192.168.1.47:8080/101/mjpg
```
```
netcam_url rtmp://192.168.1.12:1935/pi/PiCam2W47
v4l2_device /dev/video0


```
#### Client Stream config
Livestream
```
rpicam-vid -t 0 -n --framerate 30 -b 1000000 --width 970 --height 725 --codec libav --libav-format mpegts -o tcp://0.0.0.0:5556?listen=1
```
Slowstream through NGINX server
```
rpicam-vid -t 0 --width 1536 --height 864 --hdr off --nopreview \
        --exposure normal --sharpness 1.2 --contrast 1.5 --brightness 0.3 --saturation 1.0 \
        --awb indoor --denoise off --profile high --level 4.2 --codec libav \
        --libav-format flv -n --framerate 3 -b 1550000 --autofocus-mode default --inline \
        -o "rtmp://192.168.1.12/pi/PiCam2W47"
```
```
ffmpeg -f v4l2 -input_format h264 -video_size 1280x720 -framerate 30 -i /dev/video0 -vcodec copy -f flv "rtmp://192.168.1.12/pi/PiCam2W47"
```
```
text_left Kitchen
text_right ip_Address\n%Y-%m-%d\n%T-%q
text_scale 2
```

Database
```
sudo apt install libmariadb-dev libmariadb3 mariadb-client mariadb-client-core mariadb-common mariadb-server -y
```
```
sudo mariadb
```
```
GRANT ALL ON *.* TO 'motiondb'@'localhost' IDENTIFIED BY 'Mfoxboltgold' WITH GRANT OPTION;
FLUSH PRIVILEGES;
exit
```
```
mariadb -u motiondb -p
```
Mfoxboltgold

```
create database motion;
use motion;
quit;
```
```
sudo cp /usr/local/var/lib/motion/motion-dist.conf /usr/local/var/lib/motion/motion.conf
```
```
sudo nano /usr/local/var/lib/motion/motion.conf
```
```
database_type mariadb
database_dbname motion
database_host localhost
database_port 3306
database_user motiondb
database_password Mfoxboltgold
```
sudo systemctl status MariaDB


find /var/lib/motionplus -type f -mtime +2 -name '*.mkz' -execdir rm -- '{}' \;

iwconfig wlan0
sudo iw wlan0 set power_save off

