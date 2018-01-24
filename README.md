GStreamer Pipelines 
===========

Capture RTSP stream to file 
----
![pipelinePublic1](https://raw.githubusercontent.com/xmementoit/gstreamerPublicPipelines/master/pipelinePublic1.png)
```
gst-launch-1.0 rtspsrc location=rtsp://10.82.131.240:8554/h264ESVideoTest ! rtph264depay ! h264parse ! mp4mux ! filesink location=file.mp4
```

Capture RTSP stream to files splitted to 10 seconds chunks:
----
![pipelinePublic2](https://raw.githubusercontent.com/xmementoit/gstreamerPublicPipelines/master/pipelinePublic2.png)
```
gst-launch-1.0 rtspsrc location=rtsp://10.82.131.240:8554/h264ESVideoTest ! rtph264depay ! h264parse ! splitmuxsink location=file%02d.mp4 max-size-time=10000000000 
```

splitmuxsink test (overwritting max 2 files)
----
![pipelinePublic3](https://raw.githubusercontent.com/xmementoit/gstreamerPublicPipelines/master/pipelinePublic3.png)
```
gst-launch-1.0 videotestsrc ! x264enc ! h264parse ! splitmuxsink location=file%02d.mp4 max-size-time=3000000000 max-files=2
```

Get TV Channel from DVB and save it in as series of 10sec long MPEG2-TS files
----
![pipelinePublic4](https://raw.githubusercontent.com/xmementoit/gstreamerPublicPipelines/master/pipelinePublic4.png)
```
gst-launch-1.0 dvbbasebin modulation="QAM 64" trans-mode=8k bandwidth=8 frequency=682000000 code-rate-lp=AUTO code-rate-hp=2/3 guard=4  hierarchy=0 program-numbers=4170 name=src .src ! queue ! tsdemux program-number=4170 name=d  splitmuxsink location=file_%llu.ts max-size-time=1000000000 muxer=mpegtsmux name=mux   d. ! queue ! mpegvideoparse ! queue ! mux.   d. ! queue ! mpegaudioparse ! queue ! mux.audio_%u
```

Create one MPEG2-TS file from series of small chunk MPEG2-TS files
----
![pipelinePublic5](https://raw.githubusercontent.com/xmementoit/gstreamerPublicPipelines/master/pipelinePublic5.png)
```
gst-launch-1.0 splitmuxsrc location=file*.ts ! filesink location=output.ts
```

Deinterlacing video using transcoding method
----
![pipelinePublic6](https://raw.githubusercontent.com/xmementoit/gstreamerPublicPipelines/master/pipelinePublic6.png)
```
gst-launch-1.0 filesrc location=interlaced.ts ! decodebin ! videoconvert ! deinterlace ! videoconvert ! videorate ! "video/x-raw,framerate=25/1" ! avenc_mpeg2video interlaced=false gop-size=25 bitrate=100000000 ! "video/mpeg,profile=main,level=main,parsed=true" ! mpegtsmux ! filesink location=deinterlaced.ts
```

Test multiplexing audio and video to .ogg file
----
![pipeline7](https://raw.githubusercontent.com/xmementoit/gstreamerPublicPipelines/master/pipeline7.png)
```
gst-launch-1.0 videotestsrc ! queue ! videoconvert ! theoraenc ! queue ! oggmux name=mux autoaudiosrc ! queue ! audioconvert ! vorbisenc ! queue ! mux. mux. !  queue ! filesink location=test.ogg
```

Get video from camera (v4l2src) and audio from default audio source (alsasrc) and multiples them into test.ogg file
---
![pipeline8](https://raw.githubusercontent.com/xmementoit/gstreamerPublicPipelines/master/pipeline8.png)
```
gst-launch-1.0 v4l2src ! queue ! videoconvert ! theoraenc ! queue ! oggmux name=mux alsasrc ! queue ! audioconvert ! vorbisenc ! queue ! mux. mux. !  queue ! filesink location=test.ogg
```

Decode .h264 file and play video using default gstreamer player window
----
![pipeline9](https://raw.githubusercontent.com/xmementoit/gstreamerPublicPipelines/master/pipeline9.png)
```
gst-launch-1.0 filesrc location=sample.264 ! h264parse ! avdec_h264 ! videoconvert ! autovideosink
```

Decode .mp4 file and play video using default gstreamer player
----
![pipeline10](https://raw.githubusercontent.com/xmementoit/gstreamerPublicPipelines/master/pipeline10.png)
```
gst-launch-1.0 filesrc location=sample.mp4 ! qtdemux name=demux demux.audio_00 ! queue ! avdec_aac ! audioconvert ! alsasink demux.video_00 ! queue ! avdec_h264 ! videoconvert ! autovideosink
```

Play video only from .mp4 file
----
![pipeline11](https://raw.githubusercontent.com/xmementoit/gstreamerPublicPipelines/master/pipeline11.png)
```
gst-launch-1.0 filesrc location=sample.mp4 ! qtdemux name=demux ! queue ! avdec_h264 ! videoconvert ! autovideosink
```

RTP/UDP Streaming of H.264 content - Server
----
![pipeline12](https://raw.githubusercontent.com/xmementoit/gstreamerPublicPipelines/master/pipeline12.png)
```
gst-launch-1.0 -v videotestsrc ! x264enc ! rtph264pay pt=96 ! udpsink host=192.168.0.10 port=5000
```

RTP/UDP Streaming of H.264 content - Client
----
![pipeline13](https://raw.githubusercontent.com/xmementoit/gstreamerPublicPipelines/master/pipeline13.png)
```
gst-launch-1.0 -v udpsrc port=5000 caps=application/x-rtp ! rtph264depay ! avdec_h264 ! autovideosink
```

Play HLS (Http Live Streaming) URL using GStreamer pipeline
----
![pipeline14](https://raw.githubusercontent.com/xmementoit/gstreamerPublicPipelines/master/pipeline15.png)
```
gst-launch-1.0 souphttpsrc is-live=true location=http://devimages.apple.com/iphone/samples/bipbop/bipbopall.m3u8 ! hlsdemux ! decodebin ! videorate ! videoconvert ! ximagesink
```

Play Youtube Live stream as HLS video using GStreamer
----
![pipeline14](https://raw.githubusercontent.com/xmementoit/gstreamerPublicPipelines/master/pipeline14.png)
```
gst-launch-1.0 souphttpsrc is-live=true location="$(youtube-dl -f 94 -g 'https://www.youtube.com/watch?v=y60wDzZt8yg')" ! hlsdemux ! tsdemux name=mux mux. ! "video/x-h264,framerate=25/1" ! queue ! h264parse ! avdec_h264 ! videorate ! videoconvert ! ximagesink
```

Capture your desktop and save into mp4 file
----
![pipeline15](https://raw.githubusercontent.com/xmementoit/gstreamerPublicPipelines/master/pipeline15.png)
```
gst-launch-1.0 ximagesrc use-damage=0 ! video/x-raw,framerate=30/1 ! videoconvert ! x264enc ! avimux ! filesink location=out.avi
```
