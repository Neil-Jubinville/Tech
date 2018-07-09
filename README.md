# Tech
Ever bash your skull until figuring something out and don't want to forget that wonderful experience?

If I can save you some time on some nuances and complexity then I am happy to oblige.

## FFMPEG

UDP Packet Size -  Neil Jubinville

Well the other day I was setting up some screencasting to do some live streaming and was running into an issue trying to stream over my desktop and web cam from my mac to a windows 10 machine.   The windows 10 machine is running OBS and therefore I wanted to simply load VLC there to display the ffmpeg stream from the mac.

Well I spent most of a day fighting with codecs and formats mainly trying to force VLC to open a dynamic range RTP/AVP code in the stream headers.   DO NOT DO THIS, your brain will bleed.

After being certain that most of my stream settings were correct I was always getting VLC errors about the UDP Packet size being incorrect for the rtp /udp.


```bash
[00007fc833d5e380] udp stream error: 1316 bytes packet truncated (MTU was 1316)
[00007fc833d5e380] udp stream error: 1316 bytes packet truncated (MTU was 1316)
[00007fc833d5e380] udp stream error: 1316 bytes packet truncated (MTU was 1316)
[00007fc833d5e380] udp stream error: 1316 bytes packet truncated (MTU was 1316)
[00007fc833d5e380] udp stream error: 1316 bytes packet truncated (MTU was 1316)
[00007fc833d5e380] udp stream error: 1316 bytes packet truncated (MTU was 1316)
[00007fc833d5e380] udp stream error: 1316 bytes packet truncated (MTU was 1316)
[00007fc833d5e380] udp stream error: 1316 bytes packet truncated (MTU was 1316)
[00007fc833d5e380] udp stream error: 1316 bytes packet truncated (MTU was 1316)
[00007fc833d5e380] udp stream error: 1316 bytes packet truncated (MTU was 1316)
[00007fc833d5e380] udp stream error: 1316 bytes packet truncated (MTU was 1316)
[00007fc833d5e380] udp stream error: 1316 bytes packet truncated (MTU was 1316)

```

(Another tip: USE UDP,  rtp requires additional SDP/SAP services which can add complexity)

Anyhow, after severe abuse I found that I could manipulate the packet size from an obscure support post hidden in some corner of the internet.

The magic for a 1.5 sec latency (acceptable for my broadcast) stream was as follows:


```bash
#! /bin/bash

ffmpeg -f avfoundation -i "1:0" -framerate 20 -vcodec libx264 -acodec aac  -g 20 -r 100  -protocol_whitelist file,udp,tcp,http,rtp,rtmp -tune zerolatency -preset ultrafast -flush_packets 0  -b:v 2048k  -f mpegts udp://192.168.0.218:1234/?pkt_size=1316

```

The UDP url address you see was the magic with the parameter for the packet size.

### pkt_size=1316

Also note that this command you want to run from the computer that you want to stream the camera or desktop.   The UDP address represents the unicast address that the UDP packects are destined for or the computer you want to display the stream on , which in my case was the OBS broadcast system.

Cheers! Send me a tweet if this helped you out!




