📋 Detailed explanation of the options
| Option	Value	Explanation | | |
|-----| ---- | ---- |
|  nice -n -19 |		Prioritize the stream process. is the highest priority and secures computing power.| -19 |
|  -t 0	|	Infinite recording time (0 milliseconds), i.e. the stream runs until it is stopped. | |
|  --width, --height	1920, 1080	| Sets the FullHD resolution (1080p). | |
|  --nopreview 1	|	Disables local image preview, saves resources. | |
|  --low-latency 1	|	Important for low delays in the stream. | |
|  --hdr off	|	High Dynamic Range (HDR) is disabled to ensure a more authentic look. | |
|  --flush 1	|	Increases responsiveness and reduces buffering. | |
|  --buffer-count 6	|	Number of buffers used for the video frames. Optimized for streaming. | |
|  --exposure long	|	Exposure time set to 'long', often good for indoors/low light. | |
|  --sharpness 1.1, --contrast 1.2	|	Adjustment of the image properties (here slightly increased). | |
|  --codec libav		Uses the libav library for encoding and streaming. | |
|  --libav-audio 1	|	Enables audio processing by Libav. | |
|  --audio-source alsa	|	Defines ALSA as the audio source to bypass Pulse/Pipewire. | |
|  --audio-device hw:0,0		The specific audio device (card 0, device 0), determined by .arecord -l | |
|  --audio-codec aac	|	Audio codec AAC (Advanced Audio Coding), standard for streaming. | |
|  --audio-samplerate 48000	|	Sample rate for audio recording (48 kHz). | |
|  --audio-bitrate 128000	|	Audio bitrate of 128 kbps. | |
|  --libav-format flv	|	FLV (Flash Video) container format, standard for RTMP streaming. | |
|  -n	|	Disables saving to a local file. | |
|  --framerate 30	|	The desired frame rate per second. | |
|  -b 4500000	|	Video bitrate of 4.5 Mbps (4500000 bits per second). | |
|  --autofocus-mode manual	|	Autofocus is set manually. | |
|  --lens-position 0.8	|	Manual adjustment of the focus position for a firm focus. | |
|  --inline 1	|	Inserts PLC/PPS headers into the stream (important for decoding). | |
|  -o "rtmp://..."	|	The output URL to the Nginx-RTMP server. | |

###  6. Automatic start: Stream as a system service
To automatically start the stream at boot, we create a system service unit.

#### 1. Create a service file:
```
sudo nano /lib/systemd/system/stream.service
```
2. Content stream.service:

[Unit]
 
Description=ZeroCam stream service
 
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
 
WorkingDirectory=/home/axel
 
ExecStart=/usr/bin/streamfull
 
# Das streamfull.sh Skript wurde hier ohne .sh nach /usr/bin/ kopiert.
 
[Install]
 
WantedBy=multi-user.target

3. Activate and start the service:

systemctl enable stream.service
 
systemctl start stream.service
 
The stream should now run automatically every time the system starts.

7. Monitoring and troubleshooting
The system journal is used to monitor stability, especially if the stream is running as a service.

🔍 Journal Monitoring
Bash

journalctl -u stream.service -f
🟢 Stable stream output
A stable stream shows successful initializations and a continuous audio/video chain:

root@zero:/home/axel# journalctl -u stream.service -f
Okt 18 09:00:17 zero.jerabek.fi streamfull[1440]: [h264_v4l2m2m @ 0x55c1978350]  <<< v4l2_encode_init: fmt=179/0
Okt 18 09:00:17 zero.jerabek.fi streamfull[1440]: [h264_v4l2m2m @ 0x55c1978350] Using device /dev/video11
Okt 18 09:00:17 zero.jerabek.fi streamfull[1440]: [h264_v4l2m2m @ 0x55c1978350] driver 'bcm2835-codec' on card 'bcm2835-codec-encode' in mplane mode
Okt 18 09:00:17 zero.jerabek.fi streamfull[1440]: [h264_v4l2m2m @ 0x55c1978350] requesting formats: output=YU12 capture=H264
Okt 18 09:00:17 zero.jerabek.fi streamfull[1440]: Input #0, alsa, from 'hw:0,0':
Okt 18 09:00:17 zero.jerabek.fi streamfull[1440]:   Duration: N/A, start: 1760770817.363771, bitrate: 768 kb/s
Okt 18 09:00:17 zero.jerabek.fi streamfull[1440]:   Stream #0:0: Audio: pcm_s16le, 48000 Hz, 1 channels, s16, 768 kb/s
Okt 18 09:00:17 zero.jerabek.fi streamfull[1440]: Output #0, flv, to 'rtmp://192.168.178.25/zero/test':
Okt 18 09:00:17 zero.jerabek.fi streamfull[1440]:   Stream #0:0: Video: h264 (High), drm_prime(tv, bt709), 1920x1080, q=2-31, 4500 kb/s, 30 fps, 30 tbr, 1000k tbn
Okt 18 09:00:17 zero.jerabek.fi streamfull[1440]:   Stream #0:1: Audio: aac (LC), 48000 Hz, mono, fltp, 128 kb/s
🔴 Tracking Error (Unstable Stream)
Pkt tracking failure indicates instability in the stream that needs to be reset and restarted:

Oct 18 09:10:55 zero3 streamfull[2322]: [h264_v4l2m2m @ 0x55a1a21480] Pkt tracking failure: pts=101818, track[58]=101946
🔊 Audio Buffer Xruns (Typical Error)
Audio Buffer Xruns are among the most common errors in our stream logs. They are a clear indication that there are packaging errors between video and audio. At their core, they signal an overload of the system or the transmission path.

The real cause: Mostly the Wi-Fi
Although Xruns can indicate overloaded hardware, in our case the cause was very often "only" the unstable Wi-Fi connection between the Raspberry Pi and the Nginx RTMP server (or the router/Fritzbox).

root@zero3:/home/axel# journalctl -u stream.service -f
Oct 18 15:39:50 zero3 streamfull[7439]: SRGGB10_CSI2P,1640x1232/41.8515 - Score: 1000
Oct 18 15:39:50 zero3 streamfull[7439]: SRGGB10_CSI2P,1920x1080/47.5737 - Score: 1541.48
Oct 18 15:39:50 zero3 streamfull[7439]: SRGGB10_CSI2P,3280x2464/21.1941 - Score: 19329.9
Oct 18 15:39:50 zero3 streamfull[7439]: SRGGB8,640x480/103.327 - Score: 5504.81
Oct 18 15:39:50 zero3 streamfull[7439]: SRGGB8,1640x1232/41.8515 - Score: 2000
Oct 18 15:39:50 zero3 streamfull[7439]: SRGGB8,1920x1080/47.5737 - Score: 2541.48
Oct 18 15:39:50 zero3 streamfull[7439]: SRGGB8,3280x2464/21.1941 - Score: 20329.9
Oct 18 15:39:50 zero3 streamfull[7439]: Stream configuration adjusted
Oct 18 15:39:50 zero3 streamfull[7439]: [7:41:07.117053737] [7439] INFO Camera camera.cpp:1215 configuring streams: (0) 1640x1232-YUV420/Rec709 (1) 1640x1232-SRGGB10_CSI2P/RAW
Oct 18 15:39:50 zero3 streamfull[7439]: [7:41:07.117807228] [7442] INFO RPI vc4.cpp:615 Sensor: /base/soc/i2c0mux/i2c@1/imx219@10 - Selected sensor format: 1640x1232-SRGGB10_1X10/RAW - Selected unicam format: 1640x1232-pRAA/RAW
Oct 18 15:39:57 zero3 streamfull[7439]: [h264_v4l2m2m @ 0x55682da480] <<< v4l2_encode_init: fmt=179/0
Oct 18 15:39:57 zero3 streamfull[7439]: [h264_v4l2m2m @ 0x55682da480] Using device /dev/video11
Oct 18 15:39:57 zero3 streamfull[7439]: [h264_v4l2m2m @ 0x55682da480] driver 'bcm2835-codec' on card 'bcm2835-codec-encode' in mplane mode
Oct 18 15:39:57 zero3 streamfull[7439]: [h264_v4l2m2m @ 0x55682da480] requesting formats: output=YU12 capture=H264
Oct 18 15:39:57 zero3 streamfull[7439]: Input #0, alsa, from 'hw:0,0':
Oct 18 15:39:57 zero3 streamfull[7439]: Duration: N/A, start: 1760794797.840298, bitrate: 768 kb/s
Oct 18 15:39:57 zero3 streamfull[7439]: Stream #0:0: Audio: pcm_s16le, 48000 Hz, 1 channels, s16, 768 kb/s
Oct 18 15:39:57 zero3 streamfull[7439]: Output #0, flv, to 'rtmp://192.168.178.25/zero3/test':
Oct 18 15:39:57 zero3 streamfull[7439]: Stream #0:0: Video: h264 (Main), drm_prime(tv, bt709), 1640x1232, q=2-31, 3500 kb/s, 30 fps, 30 tbr, 1000k tbn
Oct 18 15:39:57 zero3 streamfull[7439]: Stream #0:1: Audio: aac (LC), 48000 Hz, mono, fltp, 64 kb/s
Oct 18 15:40:02 zero3 streamfull[7439]: [alsa @ 0x55682e1e30] ALSA buffer xrun.
Oct 18 15:40:04 zero3 streamfull[7439]: [alsa @ 0x55682e1e30] ALSA buffer xrun.
Oct 18 15:40:08 zero3 streamfull[7439]: [alsa @ 0x55682e1e30] ALSA buffer xrun.
Oct 18 15:40:13 zero3 streamfull[7439]: [alsa @ 0x55682e1e30] ALSA buffer xrun.
Oct 18 15:40:18 zero3 streamfull[7439]: [alsa @ 0x55682e1e30] ALSA buffer xrun.
Oct 18 15:40:23 zero3 streamfull[7439]: [alsa @ 0x55682e1e30] ALSA buffer xrun.
Oct 18 15:40:27 zero3 streamfull[7439]: [alsa @ 0x55682e1e30] ALSA buffer xrun.
Oct 18 15:40:29 zero3 streamfull[7439]: [alsa @ 0x55682e1e30] ALSA buffer xrun.
🛠️ Lösung und Strategie
Bitrate reduzieren: Sollte es Ihnen trotz aller Bemühungen nicht gelingen, eine stabile 100-Prozent-Verbindung zum Router herzustellen, versuchen Sie, die Datenraten von Video und/oder Audio etwas herunterzuschrauben. Dadurch verringert sich die Datenlast pro Sekunde.
Netzwerkproblem fundamental lösen: Treten die Xruns danach immer noch auf, ist eine reine Reduzierung der Bitrate nicht ausreichend. In diesem Fall muss das WLAN/WiFi-Problem grundsätzlich angegangen werden: Optimale Umpositionierung des Raspberry Pi und/oder des Routers oder der Einsatz eines WLAN-Repeaters sind die einzig dauerhaften Lösungen.
Xruns are a critical indicator. Don't ignore them because they mean your stream isn't robust enough for long-term operation.

8. 🛡️ Automated monitoring: Keep the stream running
Even with optimal configuration, external factors (such as short-term Wi-Fi fluctuations) can lead to ALSA Buffer Xruns. Before these errors make the stream completely unusable and it can no longer be retrieved from the Nginx RTMP server, proactive action should be taken.

We set up a dedicated systemd monitoring service that permanently reads the log of the actual stream () and immediately triggers a restart when the keyword "ALSA buffer xrun" is detected.stream.service

Step 1: Create the monitoring script
Create the shell script that monitors the log and performs the restart.journalctl

nano /home/pi/scripts/monitor_stream_single.sh
Contents :monitor_stream_single.sh

#!/bin/bash

SERVICE_NAME="stream.service"
LOG_FILE="/var/log/buffer_xrun_restart.log"

# Das Skript benötigt Lesezugriff auf das Journal.
# Es liest das Log des Stream-Dienstes (-u) im Follow-Modus (-f).
journalctl -u "$SERVICE_NAME" -f | while read -r line; do
    # Prüft, ob die kritische Fehlermeldung auftritt
    if echo "$line" | grep -q "ALSA buffer xrun"; then
        timestamp=$(date '+%Y-%m-%d %H:%M:%S')
        
        # Protokolliert den Neustart in einer separaten Log-Datei
        echo "[$timestamp] Detected buffer xrun. Restarting $SERVICE_NAME" >> "$LOG_FILE"
        
        # Führt den Neustart des Stream-Dienstes aus
        systemctl restart "$SERVICE_NAME"
    fi
done
Make sure the script is executable:

chmod +x /home/pi/scripts/monitor_stream_single.sh
Step 2: Create the Systemd Monitor service
Create the unit file for the monitoring service.

nano /etc/systemd/system/stream-monitor.service
Contents :stream-monitor.service

[Unit]
Description=Monitor stream.service for ALSA buffer xrun and restart on error
After=network.target

[Service]
ExecStart=/home/pi/scripts/monitor_stream_single.sh
Restart=always
RestartSec=5
StandardOutput=null
StandardError=journal
User=root

[Install]
WantedBy=multi-user.target
ParameterErklärung:
Startet das erstellte Überwachungs-Skript.
Stellt sicher, dass der Monitor selbst immer läuft, auch wenn er abstürzt.
Verhindert, dass das Skript unnötige Ausgaben ins Journal schreibt.

Wichtig, da das Skript lesen und ausführen muss.ExecStartRestart=alwaysStandardOutput=nullUser=rootjournalctlsystemctl restart

Schritt 3: Dienst aktivieren und starten
Laden Sie die Konfiguration neu, aktivieren und starten Sie den Dienst:

systemctl daemon-reload
systemctl enable stream-monitor.service
systemctl start stream-monitor.service
Ihr Raspberry Pi Zero 2 W überwacht nun proaktiv seine eigene Stream-Stabilität. Tritt ein auf, wird der Stream innerhalb weniger Sekunden neu gestartet, was die Verfügbarkeit drastisch erhöht.ALSA buffer xrun

🏁 Fazit: Die Lektion des Zero 2 W und der Wunsch an die Entwickler
The journey to the stable FullHD livestream from the Raspberry Pi Zero 2 W was a lesson in patience and system understanding. The most important finding is that we are dealing with a system that hardly tolerates buffers for micro-interruptions in the data flow.

The greatest frustrations arise from the fact that the sources of error are not obvious. The best example is the Wi-Fi speed: mathematically, 30 Mbit/s should be more than enough for a 4 Mbit stream. In practice, however, it has been shown that without a stable 72 Mbit connection, the stream inevitably tips over – a mathematical fallacy that can lead to hours of troubleshooting in the wrong place (e.g. CPU or swap).

Initially, all latent sources of error together made the stream unstable. It was only by consistently optimizing every single link in the chain – from eliminating the swap to the correct governor setting to uncompromising network performance – that we were able to achieve robust 24/7 operations. The small Pi Zero 2 W usually does not recover during short dropouts in the WLAN; the stream is then irrevocably broken.

Finally, there is only one wish that would make the streams even more reliable:

We wish that the rpicam-apps team will finally integrate a reasonable method of synchronization regarding video and audio sync when initializing the stream!

Until then, we hope that our accumulated experience will help you set up your perfect livestream on the Raspberry Pi Zero 2 W quickly and stably.

Appendix: CPU Governor Shell Script (cpu-settings.sh)
Bash

#!/bin/bash
# Pfad zum CPU-Governor-Verzeichnis
GOVERNOR_PATH="/sys/devices/system/cpu/cpufreq/policy0/scaling_governor"

# Funktion, um den aktuellen Governor anzuzeigen
show_governor() {
    if [ -f "$GOVERNOR_PATH" ]; then
        current_governor=$(cat "$GOVERNOR_PATH")
        echo "Aktueller CPU-Governor: $current_governor"
    else
        echo "Fehler: Konnte den Pfad zum Governor-Verzeichnis nicht finden."
        exit 1
    fi
}

# Funktion, um den Governor zu ändern
set_governor() {
    local new_governor=$1
    echo "Ändere Governor zu '$new_governor'..."
    if [ -f "$GOVERNOR_PATH" ]; then
        echo "$new_governor" | sudo tee "$GOVERNOR_PATH" > /dev/null
        if [ $? -eq 0 ]; then
            echo "Erfolgreich geändert."
        else
            echo "Fehler: Konnte den Governor nicht ändern. Möglicherweise fehlen root-Rechte."
        fi
    else
        echo "Fehler: Konnte den Pfad zum Governor-Verzeichnis nicht finden."
        exit 1
    fi
}

# Hauptlogik des Skripts
case "$1" in
    "status")
        show_governor
        ;;
    "ondemand")
        set_governor "ondemand"
        ;;
    "performance")
        set_governor "performance"
        ;;
    "powersave")
        set_governor "powersave"
        ;;
    "schedutil")
        set_governor "schedutil"
        ;;
    *)
        echo "Verwendung: $0 [status|ondemand|performance|powersave|schedutil]"
        echo "   status: Zeigt den aktuellen Governor an."
        echo " ondemand: Stellt den Governor auf 'ondemand'."
        echo "performance: Stellt den Governor auf 'performance'."
        echo "powersave: Stellt den Governor auf 'powersave'."
        echo "schedutil: Stellt den Governor auf 'schedutil'."
        ;;
esac
Appendix 2 – Nginx-RTMP Server Example config: (with simultaneous HLS stream)
?
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
 
        # HLS Streams für mevo
        location /hlsmevo/ {
            root /var/hls;
            types {
                application/vnd.apple.mpegurl m3u8;
                video/mp2t ts;
            }
            add_header Cache-Control no-cache;
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
 
        # HLS Streams für zero2
        location /hlszero2/ {
            root /var/hls;
            types {
                application/vnd.apple.mpegurl m3u8;
                video/mp2t ts;
            }
            add_header Cache-Control no-cache;
        }
 
        # HLS Streams für zero3
        location /hlszero3/ {
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
 
        application mevo {
            live on;
            record off;
 
            hls on;
            hls_path /var/hls/hlsmevo;
            hls_fragment 8;
            hls_playlist_length 16;
            hls_type live;
            sync 100ms;
            hls_cleanup on;
        }
 
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
 
        application zero2 {
            live on;
            record off;
 
            hls on;
            hls_path /var/hls/hlszero2;
            hls_fragment 8;
            hls_playlist_length 16;
            hls_type live;
            sync 100ms;
            hls_cleanup on;
        }
 
        application zero3 {
            live on;
            record off;
 
            hls on;
            hls_path /var/hls/hlszero3;
            hls_fragment 8;
            hls_playlist_length 16;
            hls_type live;
            sync 100ms;
            hls_cleanup on;
        }
    }
}
Post navigation
