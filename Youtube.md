```
#!/bin/bash
nice -n -19 rpicam-vid -t 0 --width 1920 --height 1080 \
--nopreview 1 --low-latency 1 --hdr off --flush 1 \
--buffer-count 6 --exposure long --sharpness 1.1 --contrast 1.2 \
--brightness 0.0 --saturation 1.0 --ev +1.0 --awb auto \
--profile high --level 4.2 --codec libav --libav-audio 1 \
--audio-source alsa --audio-device hw:0,0 --audio-channels 1 \
--audio-codec aac \
--audio-samplerate 24000 --audio-bitrate 12800 --libav-format flv \
-n --framerate 10 -b 750000 --autofocus-mode manual --lens-position 0.8 \
--denoise auto --autofocus-window 0.25,0.25,0.5,0.5 --inline 1 \
-o "rtmp://a.rtmp.youtube.com/live2/eu6j-71px-cfvp-qsbe-f1hm"

```
