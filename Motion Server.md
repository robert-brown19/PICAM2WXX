## Creating Motion Server

Use PI Imager to install latest full verion of PI-OS


Building a deb package
A script has also been written to allow users to create their own deb package from either a tagged release or the most current master branch of the code. Download the script by using the following command. wget https://raw.githubusercontent.com/Motion-Project/motion-packaging/master/build.sh
Review the script and specify the following few optional parameters. Username, EmailAddress, branch, install, arch. If the parameters are not provided, the script will create defaults.
Once the script has been run and the deb file is created, install it as described above.




Add require Motion libraries

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
make install

```
Add supporting STIG/SCAP required libraries
```
sudo apt install opensc libpam-pkcs11 auditd ufw libpam-pwquality apparmor vlock aide chrony -y
```
NGINX Install
```
sudo apt install nginx-common nginx-core libnginx-mod-rtmp
```
Append to /etc/ngnix/nginx.conf

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



libcam_device camera0

Testing step
libcamera-hello
libcamerify motionplus

## Configuration
Default Dir /usr/local/var/lib/motion
```
sudo mkdir /usr/local/var/lib/motion/conf.d
```
```
sudo nano /usr/local/var/lib/motion/motion.conf
```
```
sudo cp /usr/local/var/lib/motion/camera1-dist.conf /usr/local/var/lib/motion/conf.d/camera.conf
sudo nano /usr/local/var/lib/motion/conf.d/camera.conf



netcam_url http://10.49.134.111:8080/101/mjpg

text_left Kitchen
text_right ip_Address\n%Y-%m-%d\n%T-%q
text_scale 2

/etc/group   add motion to adm group
/etc/logrotate.d/motionplus  add su motion adm

/etc/motionplus/camera1.conf  add under Camera target_dir /var/lib/motionplus

Database
sudo apt install libmariadb-dev libmariadb3 mariadb-client mariadb-client-core mariadb-common mariadb-server -y

sudo mariadb
GRANT ALL ON *.* TO 'motiondb'@'localhost' IDENTIFIED BY 'Mfoxboltgold' WITH GRANT OPTION;
FLUSH PRIVILEGES;
exit

mariadb -u motiondb -p
Mfoxboltgold
create database motionplus;
use motionplus;
quit;

database_type mariadb
database_dbname motionplus
database_host localhost
database_port 3306
database_user motiondb
database_password Mfoxboltgold

sudo systemctl status MariaDB

netcam_url http://10.49.134.108:8080/101/mjpg

netcam_url http://10.49.134.109:8080/101/mjpg

netcam_url http://10.49.134.110:8080/101/mjpg

netcam_url http://10.49.134.111:8080/101/mjpg

netcam_url http://10.49.134.124:8080/101/mjpg

netcam_url http://10.49.134.125:8080/101/mjpg

netcam_url http://10.49.134.147:8080/101/mjpg

netcam_url http://10.49.134.148:8080/101/mjpg

netcam_url http://10.49.134.149:8080/101/mjpg


W: http://raspbian.raspberrypi.com/raspbian/dists/bookworm/InRelease: Key is stored in legacy trusted.gpg keyring (/etc/apt/trusted.gpg), see the DEPRECATION section in apt-key(8) for details.


find /var/lib/motionplus -type f -mtime +2 -name '*.mkz' -execdir rm -- '{}' \;

iwconfig wlan0
sudo iw wlan0 set power_save off

