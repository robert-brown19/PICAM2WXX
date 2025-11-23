# CAM Config 11/23/25

## Trixie

- [ ] OS Task  
```
sudo apt update \&\& sudo apt full-upgrade -y
```

- [ ] WiFi Task
```
sudo nmcli con add type wifi con-name PodZero1 ssid PodZero1 802-11-wireless-security.key-mgmt wpa-psk
sudo nmcli --ask con up id "PodZero1"
sudo nmcli con show

sudo nmcli con edit PodZero1
nmcli> set connection.autoconnect-priority 10
nmcli> save persistent
```

- [ ] Package Task
```
sudo apt purge rpicam-apps-lite
sudo apt install ffmpeg rpicam-apps libavcodec-dev libavdevice-dev libavformat-dev libswresample-dev -y
```

- [ ] Stream Task  

| tabel | x |  x |
|  ---  | ---  |---|  
| Cam v2 | 1920 x 1080p47 | 1640 × 1232p41 and 640 × 480p206 |  
| Cam v3 | 2304 × 1296p56 | 2304 × 1296p30 HDR, 1536 × 864p120 |  

sudo nano streamslow.sh

\#!/bin/bash

rpicam-vid -t 0 --width 1640 --height 1232 --hdr off --nopreview  
--exposure normal --sharpness 1.2 --contrast 1.5 --brightness 0.3 --saturation 1.0  
--awb indoor --denoise off --profile high --level 4.2 --codec libav  
--libav-format flv -n --framerate 3 -b 1550000 --autofocus-mode default --inline  
-o "rtmp://192.168.4.48/pi/PiCam51"


- [ ] Create Service Task

sudo nano /etc/systemd/system/streamslow.service

\[Unit]
Description=Start Camera Script
After=multi-user.target
\[Service]
ExecStart=/home/fox-admin/streamslow.sh
Type=idle
\[Install]
WantedBy=multi-user.target



sudo chmod 644 /etc/systemd/system/streamslow.service

Finally, tell systemd to run it at boot:

sudo systemctl enable streamslow.service

sudo systemctl stop streamslow.service

