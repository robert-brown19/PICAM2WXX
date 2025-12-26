# CAM Config 11/23/25

## Trixie

- [ ] OS Task  
```
sudo apt update && sudo apt full-upgrade -y
```

- [ ] WiFi Task
```
sudo nmcli con add type wifi con-name PodZero1 ssid PodZero1 802-11-wireless-security.key-mgmt wpa-psk
```
```
sudo nmcli --ask con up id "PodZero1"
```
```
sudo nmcli con show
```
```
sudo nmcli con edit PodZero1
```
```
nmcli> set connection.autoconnect-priority 10
nmcli> save persistent
```

- [ ] Package Task
```
sudo apt purge rpicam-apps-lite -y
```
```
sudo apt install ffmpeg rpicam-apps libavcodec-dev libavdevice-dev libavformat-dev libswresample-dev -y
```
```
sudo apt autoremove
```

- [ ] Stream Task  

| Table | x |  x | x |
|  ---  | ---  | --- |  --- |
| Camera Module v2 | 1920 x 1080p47 | 1640 × 1232p41 | 640 × 480p206 |  
| Camera Module v3 | 2304 × 1296p56 | 2304 × 1296p30 HDR | 1536 × 864p120 |  
| Camera Module v3 Wide | 2304 × 1296p56 | 2304 × 1296p30 HDR | 1536 × 864p120 |
| GS Camera Module | 1456 × 1088p60 |  | |

```
sudo nano streamslow.sh
```
```
#!/bin/bash

rpicam-vid -t 0 --width 1536 --height 864 --hdr off  --nopreviewv 1\
        --exposure long --sharpness 1.2 --contrast 1.4 --brightness 0.1 --saturation 1.0 \
        --awb auto --denoise auto --profile high --level 4.2 --codec libav \
        --av-sync 0 --libav-format flv -n --framerate 3 -b 1550000 --autofocus-mode manual --lens-position 0.5 \
        --autofocus-window 0.25,0.25,0.5,0.5 --inline \
        -o "rtmp://192.168.1.12/pi/PiCam2W51"
```
### streamsteady.sh
```
sudo nano streamsteady.sh
```
```
#!/bin/bash
nice -n -19 rpicam-vid -t 0 --width 1920 --height 1080 \
--nopreview 1 --low-latency 1 --hdr off --flush 1 \
--buffer-count 6 --exposure long --sharpness 1.1 --contrast 1.2 \
--brightness 0.0 --saturation 1.0 --ev +1.0 --awb auto \
--profile high --level 4.2 --codec libav --libav-audio 1 \
--audio-source alsa --audio-device hw:0,0 --audio-channels 1 \
--audio-codec aac \
--audio-samplerate 48000 --audio-bitrate 128000 --libav-format flv \
-n --framerate 30 -b 4500000 --autofocus-mode manual --lens-position 0.8 \
--denoise auto --autofocus-window 0.25,0.25,0.5,0.5 --inline 1 \
-o "rtmp://192.168.1.12/pi/PiCam2W51"
```
- [ ] Create Service Task
```
sudo nano /etc/systemd/system/streamsteady.service
```
```
[Unit]
Description=Start Camera stream service
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
WorkingDirectory=/home/fox-admin
ExecStart=/home/fox-admin/streamsteady.sh

[Install]
WantedBy=multi-user.target
```

```
sudo chmod 644 /etc/systemd/system/streamslow.service
```
Finally, tell systemd to run it at boot:
```
sudo systemctl enable streamslow.service
```
```
sudo systemctl stop streamslow.service
```
