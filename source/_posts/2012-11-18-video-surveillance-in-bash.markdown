---
layout: post
title: "Video Surveillance... in Bash"
date: 2012-11-18 00:47
comments: true
categories: ["*nix", Shell, Coding]
---

My parents recently asked me to install an IP video surveillance camera at their
house (for reasons). Since I already have a Linux server there running 24/7, it
was only natural that I somehow set it up to record the camera's video feed (mp4
over RTSP). After a bit of googling I found [ZoneMinder][1], which looked like
it did everything I needed. However, after wasting half a day fiddling with it,
I realized that it is, although very complex and feature-rich, not up to my
needs: no matter how you configure it, it will convert the video feed into
thousands of JPEG files and store them randomly on the filesystem. It also uses
a MySQL database to store its information (so extra dependencies), viewing the
camera live is very slow (refreshes a static image every 4-5 seconds), replaying
a recording needs the Java plug-in in your browser (which I don't have on
Ubuntu) and you have to "export" a movie clip (i.e. it has to convert all the
images it saved back to a video). What a waste, but I can see how this solution
could be the most compatible with all possible environments (camera types, OSes,
etcetera). I was unable to set up authentication or a maximum total file size,
either.

Being fed up with ZoneMinder, I ditched it and decided to try and hack together
something of my own. After fiddling for the next half of the day with `ffmpeg`,
I came up with a viable solution, in bash.

``` bash bashsurv.sh https://gist.github.com/4100939 View Gist
#!/bin/bash

OUTPUT_DIR="/var/www/bashsurv"
FFMPEG_INPUT_FLAGS="-rtsp_transport udp"
FFMPEG_SOURCE="rtsp://192.168.1.123/video.mp4"
FFMPEG_OUTPUT_FLAGS="-r 20 -acodec libspeex"
FFMPEG_OUTPUT_EXT="ogv"
CLIP_LENGTH=60 # seconds
KEEP_FILES_FOR=5 # minutes

while [ true ]; do
  ffmpeg -t $CLIP_LENGTH $FFMPEG_INPUT_FLAGS -i $FFMPEG_SOURCE \
    $FFMPEG_OUTPUT_FLAGS $OUTPUT_DIR/$(date +%F.%T).$FFMPEG_OUTPUT_EXT
  find $OUTPUT_DIR/ -type f -mmin +$KEEP_FILES_FOR -delete
done
```

Unfortunately, I was not able to just copy the codec, which would have been
optimal in terms of resource (CPU/memory) usage on my server, but it doesn't
use up *that* much. Now I can set `OUTPUT_DIR` to somewhere in my `/var/www`,
set up authentication on it via `.htaccess` and be done with it. Dad can now
easily view the recordings using his browser, and if he really needs to view the
live stream I can just bookmark the RTSP stream in VLC for him or something.

Bottom line: K.I.S.S.

[0]: http://www.tp-link.com/en/products/details/?model=TL-SC3130G
[1]: http://www.zoneminder.com/
