---
layout: post
title: "Streaming video and audio of an USB webcam to multiple users of a website with SSL, basic authentication and in-video timestamps (FFmpeg, RTMP, NGINX, HLS and MPEG-Dash)"
categories: [IT]
image: images/2020-12-31-streaming-video-and-audio-of-an-usb-webcam-to-multiple-users-of-a-website-with-ssl-basic-authentication-and-invideo-timestamps-ffmpeg-rtmp-nginx-hls-mpeg-dash/webcam_small.jpg
excerpt: "Having an USB webcam connected to a Raspberry Pi 4, we're going to use FFmpeg to securely (SSL and secret-key authentication) stream its video and audio (with CCTV-style timestamps) to a containerized NGINX server over RTMP/RTMPS protocol, and then use this same server to broadcast the stream to multiple users using both Apple HLS and MPEG-Dash. Users will be able to watch the stream on the browser, Android, iOS, Linux, Windows and MacOS."
---

[![GIF]({{ site.baseurl }}/images/2020-12-31-streaming-video-and-audio-of-an-usb-webcam-to-multiple-users-of-a-website-with-ssl-basic-authentication-and-invideo-timestamps-ffmpeg-rtmp-nginx-hls-mpeg-dash/waving.gif)]({{ site.baseurl }}/images/2020-12-31-streaming-video-and-audio-of-an-usb-webcam-to-multiple-users-of-a-website-with-ssl-basic-authentication-and-invideo-timestamps-ffmpeg-rtmp-nginx-hls-mpeg-dash/waving.gif)

(Please don't mind the mess)

Having an USB webcam connected to a Raspberry Pi 4, we're going to use FFmpeg to securely (SSL and secret-key authentication) stream its video and audio (with CCTV-style timestamps) to a containerized NGINX server over RTMP/RTMPS protocol, and then use this same server to broadcast the stream to multiple users using both [Apple HLS](https://en.wikipedia.org/wiki/HTTP_Live_Streaming) and [MPEG-Dash](https://en.wikipedia.org/wiki/Dynamic_Adaptive_Streaming_over_HTTP). Users will be able to watch the stream on the browser, Android, iOS, Linux, Windows and MacOS.

[![Webcam]({{ site.baseurl }}/images/2020-12-31-streaming-video-and-audio-of-an-usb-webcam-to-multiple-users-of-a-website-with-ssl-basic-authentication-and-invideo-timestamps-ffmpeg-rtmp-nginx-hls-mpeg-dash/webcam.jpg)]({{ site.baseurl }}/images/2020-12-31-streaming-video-and-audio-of-an-usb-webcam-to-multiple-users-of-a-website-with-ssl-basic-authentication-and-invideo-timestamps-ffmpeg-rtmp-nginx-hls-mpeg-dash/webcam.jpg)

[![Architecture layout]({{ site.baseurl }}/images/2020-12-31-streaming-video-and-audio-of-an-usb-webcam-to-multiple-users-of-a-website-with-ssl-basic-authentication-and-invideo-timestamps-ffmpeg-rtmp-nginx-hls-mpeg-dash/layout.png)]({{ site.baseurl }}/images/2020-12-31-streaming-video-and-audio-of-an-usb-webcam-to-multiple-users-of-a-website-with-ssl-basic-authentication-and-invideo-timestamps-ffmpeg-rtmp-nginx-hls-mpeg-dash/layout.png)

This is useful if you're willing to have any video source (file or live feed) streaming on the internet or network, maybe also host the stream on your own website (embedded as HTML) to display public events/places, all-sky-cameras, CCTV, science experiments, etc. Or anything that won't match with some of the major streaming platform policies. Or something that's designed to broadcast 24/7. The cases are numerous. Whatever the use-case be, I have applied here simple but efficient methods to limit who can stream, who can watch and also to end-to-end encrypt everything. In this article I'm going to demonstrate how I configured it all using common "of-the-shelf" tools, and I hope this helps you.

But before we begin, note that using this configuration I'm experiencing about 50 seconds of delay between what's recorded and what's played with all devices communicating over a 54mpbs Wifi network. But the biggest bottleneck I assume it's NGINX processing the stream.

## Preparing the NGINX Server

The most important part of this setup is the NGINX server. This server is going to receive the media stream from the Raspberry Pi, decrypt it, authenticate and serve it as both HLS and Dash. However NGINX won't support RTMP media stream by default, we'll have to compile it with the [RTMP module](https://github.com/arut/nginx-rtmp-module/wiki/Directives). This process is straight-forward but tedious, so I've built a [Docker image](https://github.com/fschuindt/nginx_rtmp_hls_dash) to do that for us. In case you want to do it manually for some reason you can follow the commands of the Dockerfile:

<script src="https://gist.github.com/fschuindt/dd307414bc7ecea00a837309160c9f69.js"></script>

For you to set this container up, first clone the Docker image repository:
```
$ git clone https://github.com/fschuindt/nginx_rtmp_hls_dash.git
```

Then `cd` into it:
```
cd nginx_rtmp_hls_dash
```

Now you will need [Docker](https://docs.docker.com/get-docker/) and [Docker Compose](https://docs.docker.com/compose/install/) installed on your machine. Installing them is pretty simple, so take a minute to do so if you don't have them already.

Build the image:
```
$ docker-compose build broadcaster
```

And spawn up the NGINX server:
```
$ docker-compose up broadcaster
```

The server is now ready to accept streams and to start broadcasting them.

This will leave NGINX running attached to the `4080`, `4443`, `4080` and `4936` ports on the Docker host machine. Check the [repository documentation](https://github.com/fschuindt/nginx_rtmp_hls_dash#running) for more information on those ports. Or better yet, take a look at the `nginx.conf` file:

<script src="https://gist.github.com/fschuindt/e1691faa591c406beb4c909fa00a02e2.js"></script>

You can change this file as you see fit, just be sure to check the ["Warnings"](https://github.com/fschuindt/nginx_rtmp_hls_dash#nginx-rtmprtmps-to-hls-and-mpeg-dash-media-stream-broadcaster) section of the repository documentation.

As you can see on the `nginx.conf`, we have a rudimentar secret-key based authentication. It's enough for the purpose of this article and for the most simple usage of media streaming. Mind though that it only makes sense to use this authentication over SSL/TLS encryption, because the keys will be sent as plain text otherwise.

The configuration also describes a SSL and a plain-text endpoint for each connection. So you can choose between encrypted and unencrypted on all steps.

## Sending webcam audio and video to the NGINX server

Now that the server is up and running, we need to SSH into the Raspberry Pi (or any computer with an USB webcam). In my case the Pi has a generic USB webcam connected to it. Our goal is to find this webcam device and to use [FFmpeg](https://ffmpeg.org/) to start a RTPM and RTPMS audio/video stream to the NGINX server over the network. Be sure to have FFmpeg installed on your Pi/computer, this should be simple to do.

Also mind that here I'm going to be describing steps for a Linux computer. Although FFmpeg is multi-platform, the way the video data input is harvested may change depending on the OS. I'm going to use [Video4Linux](https://en.wikipedia.org/wiki/Video4Linux). On the repository `README.md` you will find a more simple example for streaming `.mp4` files.

The first step is to find the webcam device:
```
$ v4l2-ctl --list-devices

bcm2835-codec-decode (platform:bcm2835-codec):
	/dev/video10
	/dev/video11
	/dev/video12
	/dev/media0

bcm2835-isp (platform:bcm2835-isp):
	/dev/video13
	/dev/video14
	/dev/video15
	/dev/video16
	/dev/media1

GENERAL WEBCAM: GENERAL WEBCAM (usb-0000:01:00.0-1.3):
	/dev/video0
	/dev/video1
	/dev/media2
```

This output tells me that `/dev/video0` is the webcam.

This webcam have a built-in microphone and we also want to stream the audio, so we need to find its audio interface:

```
$ arecord -L

null
    Discard all samples (playback) or generate zero samples (capture)
samplerate
    Rate Converter Plugin Using Samplerate Library
speexrate
    Rate Converter Plugin Using Speex Resampler
jack
    JACK Audio Connection Kit
oss
    Open Sound System
pulse
    PulseAudio Sound Server
speex
    Plugin using Speex DSP (resample, agc, denoise, echo, dereverb)
upmix
    Plugin for channel upmix (4,6,8)
vdownmix
    Plugin for channel downmix (stereo) with a simple spacialization
default
    Default ALSA Output (currently PulseAudio Sound Server)
usbstream:CARD=Headphones
    bcm2835 Headphones
    USB Stream Output
sysdefault:CARD=WEBCAM
    GENERAL WEBCAM, USB Audio
    Default Audio Device
front:CARD=WEBCAM,DEV=0
    GENERAL WEBCAM, USB Audio
    Front output / input
usbstream:CARD=WEBCAM
    GENERAL WEBCAM
    USB Stream Output
```

It's the one set as the default: `sysdefault:CARD=WEBCAM`.

We also need to discover the supported resolutions for the webcam:
```
$ ffmpeg  -hide_banner -f video4linux2 -list_formats all -i /dev/video0

[video4linux2,v4l2 @ 0xaaaac7504320] Compressed:       mjpeg :          Motion-JPEG : 1920x1080 1440x1080 1280x720 800x600 800x480 720x480 640x480 640x360 480x270 320x240 176x144
[video4linux2,v4l2 @ 0xaaaac7504320] Raw       :     yuyv422 :           YUYV 4:2:2 : 1280x720 800x600 800x480 720x480 640x480 640x360 480x270 320x240 176x144
```

I think `1280x720` is a good choice for this experiment.

*Now a quick side note:* I decided to play with webcam streaming to help me in a weather project for astronomy data acquisition, so for me a CCTV-style timestamp and the ability to write text on the video was essential. Luckily FFmpeg has it all covered (check the `drawtext` option on the next command).

As my laptop's network IP (running the NGINX) is `192.168.1.192`, this is how I composed my RTMPS webcam streaming with in-video timestamps, from the Raspberry Pi:
```
ffmpeg \
    -f video4linux2 -framerate 25 -video_size 1280x720 -i /dev/video0 \
    -f alsa -ac 2 -i sysdefault:CARD=WEBCAM \
    -c:v libx264 -b:v 1600k -preset ultrafast \
    -x264opts keyint=50 -g 25 -pix_fmt yuv420p \
    -c:a aac -b:a 128k \
    -vf "drawtext=fontfile=/usr/share/fonts/dejavu/DejaVuSans-Bold.ttf: \
text='CLOUD DETECTION CAMERA 01 UTC-3 %{localtime\:%Y-%m-%dT%T}': fontcolor=white@0.8: fontsize=16: x=10: y=10: box=1: boxcolor=black: boxborderw=6" \
    -f flv "rtmps:192.168.1.192:4936/live/cam1?streamkey=5f3e32f3bad0"
```

Note the `rtmps:192.168.1.192:4936/live/cam1?streamkey=5f3e32f3bad0` address, sending the RTMPS to the NGINX server.

Be sure to read the repository [`README.md`](https://github.com/fschuindt/nginx_rtmp_hls_dash#nginx-rtmprtmps-to-hls-and-mpeg-dash-media-stream-broadcaster) to learn more about the address format and the stream-key.

After running it, the RPi4 is already streaming the webcam audio and video to the NGINX server.

## Watching it: HLS or MPEG-Dash?

Many articles go deep into the difference between the two. My take is that unless you have a reason, use MPEG-Dash over HLS, as it's newer and "smart" enough to adapt the video quality to match the viewer's connection bandwidth.

So from now on in this article I'm going to be covering only MPEG-Dash examples. If you want to know how to watch HLS, check the [repository documentation](https://github.com/fschuindt/nginx_rtmp_hls_dash#watching-play-video-stream).

## Watching the stream on VLC (Windows, Linux and MacOS)

If you have [VLC](https://www.videolan.org/vlc/index.html) installed, you can open it and press `Ctrl + n` or `Cmd + n`. On the network address input, enter:
```
https://192.168.1.192:4443/dash/cam1.mpd?watchkey=16356b9f
```

Replace `192.168.1.192` with your NGINX server address.

Hit enter, accept the certificate issue (it's a self signed SSL certificate) and wait for the stream to begin:

[![VLC on Linux]({{ site.baseurl }}/images/2020-12-31-streaming-video-and-audio-of-an-usb-webcam-to-multiple-users-of-a-website-with-ssl-basic-authentication-and-invideo-timestamps-ffmpeg-rtmp-nginx-hls-mpeg-dash/vlc_on_linux.png)]({{ site.baseurl }}/images/2020-12-31-streaming-video-and-audio-of-an-usb-webcam-to-multiple-users-of-a-website-with-ssl-basic-authentication-and-invideo-timestamps-ffmpeg-rtmp-nginx-hls-mpeg-dash/vlc_on_linux.png)

## Watching the stream on mobile (Android and iOS)

For that any MPEG-Dash (and HLS) compatible mobile client will work. I don't have an iOS device here to test, but I'm sure there's plenty of clients to choose. For Android I'm going to use the [ExpressPlayer](https://play.google.com/store/apps/details?id=com.intertrust.ExpressPlayer&hl=en&gl=US).

For some reason this app does not implement HTTPS (shame!), so I'm going to use HTTP in this example.

Open the app, select "CUSTOM INPUT", then on "Media/MS3 URL ..." add:
```
http://192.168.1.192:4080/dash/cam1.mpd?watchkey=16356b9f
```

Click "Play" and select "WV-DASH-M4F":

[![Playing on Android]({{ site.baseurl }}/images/2020-12-31-streaming-video-and-audio-of-an-usb-webcam-to-multiple-users-of-a-website-with-ssl-basic-authentication-and-invideo-timestamps-ffmpeg-rtmp-nginx-hls-mpeg-dash/playing_on_android.jpg)]({{ site.baseurl }}/images/2020-12-31-streaming-video-and-audio-of-an-usb-webcam-to-multiple-users-of-a-website-with-ssl-basic-authentication-and-invideo-timestamps-ffmpeg-rtmp-nginx-hls-mpeg-dash/playing_on_android.jpg)

## Watching the stream on the web (Google Chrome, Firefox, etc.)

There are two ways I know of for playing MPEG-Dash on browsers: [Dash.js](https://reference.dashif.org/dash.js/latest/samples/index.html) and [Google's Shaka Player](https://github.com/google/shaka-player), both JavaScript implementations.

Here I'm going to display Google's Shaka in action.

Also, as we're using a self signed certificate, Shaka will complain about not being able to verify the authority, so here too we'll need to use HTTP instead of HTTPS. But this should be fixed if ever on production, as proper certificates will be used.

`index.html`:
```html
<!DOCTYPE html>
<html>
  <head>
    <script src="js/shaka-player.compiled.js"></script>
    <script src="js/app.js"></script>
  </head>
  <body>
    <video id="video" width="1280" controls autoplay></video>
  </body>
</html>
```

`app.js` (taken from [here](http://halcyon.ch/dash-tutorial-2-display-dash-stream/))
```js
const manifestUri =
      'http://192.168.1.192:4080/dash/bbb.mpd?watchkey=16356b9f';

function initApp() {
  shaka.polyfill.installAll();

  if (shaka.Player.isBrowserSupported()) {
    initPlayer();
  } else {
    console.error('Browser not supported.');
  }
}

async function initPlayer() {
  const video = document.getElementById('video');
  const player = new shaka.Player(video);

  window.player = player;
  player.addEventListener('error', onErrorEvent);

  try {
    await player.load(manifestUri);
  } catch (e) {
    onError(e);
  }
}

function onErrorEvent(event) {
  onError(event.detail);
}

function onError(error) {
  console.error('Error code', error.code, 'object', error);
}

document.addEventListener('DOMContentLoaded', initApp);
```

And it works like a charm, it's the best player I've tested so far.

[![Thumbs up]({{ site.baseurl }}/images/2020-12-31-streaming-video-and-audio-of-an-usb-webcam-to-multiple-users-of-a-website-with-ssl-basic-authentication-and-invideo-timestamps-ffmpeg-rtmp-nginx-hls-mpeg-dash/thumbs_up.png)]({{ site.baseurl }}/images/2020-12-31-streaming-video-and-audio-of-an-usb-webcam-to-multiple-users-of-a-website-with-ssl-basic-authentication-and-invideo-timestamps-ffmpeg-rtmp-nginx-hls-mpeg-dash/thumbs_up.png)

## Bonus: We don't need NGINX for a single viewer

If your goal is to have a single computer consuming the video stream, eg.: RPi sends webcam video to a desktop computer to be recorded. Then you don't need a NGINX server, you can stream directly from the RPi to the desktop computer using just FFmpeg and VLC over the much simpler [RTP protocol](https://en.wikipedia.org/wiki/Real-time_Transport_Protocol).

Some comments on that:
- You will need to decide between playback or record, I failed to perform both on VLC without crashing.
- The record-and-play delay is a lot smaller, around 10 seconds on my setup.
- No encryption nor authentication.

First, let's name the machines:
```
RPi - 192.168.1.193
Desktop - 192.168.1.155
```

Start to stream the webcam video, from the RPi:
```
ffmpeg \
    -f video4linux2 -framerate 25 -video_size 1280x720 -i /dev/video0 \
    -f alsa -ac 2 -i sysdefault:CARD=WEBCAM \
    -c:v libx264 -b:v 1600k -preset ultrafast \
    -x264opts keyint=50 -g 25 -pix_fmt yuv420p \
    -c:a aac -b:a 128k \
    -vf "drawtext=fontfile=/usr/share/fonts/dejavu/DejaVuSans-Bold.ttf: \
text='CLOUD DETECTION CAMERA 01 UTC-3 %{localtime\:%Y-%m-%dT%T}': fontcolor=white@0.8: fontsize=16: x=10: y=10: box=1: boxcolor=black: boxborderw=6" \
    -f rtp_mpegts "rtp://192.168.1.155:5000?ttl=2"
```

Note that I'm pointing to the Desktop machine even though there's no server running there. `rtp://192.168.1.155:5000?ttl=2`. I've also chosen the port `5000`, but that's up to you.

Now, on the Desktop computer open VLC and press `Ctrl + n` or `Cmd + n` to open the network stream dialog. And on the address input enter: `rtp://192.168.1.155:5000`. Then press Enter to start watching the stream.

That's correct, we're entering the same IP of the Desktop computer. That's the computer which is receiving the stream.

## That's it

Video streaming is definitely fun. I can see many projects where this will be useful. I'm also feeling happy after learning how to set up these configurations and I hope you have enjoyed it as well. Feel free to contact me if you're facing issues or just if you want to chat.

See you!
