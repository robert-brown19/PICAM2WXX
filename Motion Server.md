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
sudo apt install nginx-common nginx-core libnginx-mod-rtmp
```
Append to /etc/nginx/nginx.conf
```
sudo nano /etc/nginx/nginx.conf
```
```
rtmp {  
server {  
    listen 1935;  
    timeout 60s;  
    notify_method post;  
    chunk_size 4096;  

application pi {  
live on;  
record off;  
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


```
netcam_url http://10.49.134.111:8080/101/mjpg

text_left Kitchen
text_right ip_Address\n%Y-%m-%d\n%T-%q
text_scale 2
```

Database
sudo apt install libmariadb-dev libmariadb3 mariadb-client mariadb-client-core mariadb-common mariadb-server -y
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
create database motionplus;
use motionplus;
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

