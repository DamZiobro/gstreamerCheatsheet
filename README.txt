#==============================================================================================
#pipelinePublic1 
#Capture RTSP stream to file 
gst-launch-1.0 rtspsrc location=rtsp://10.82.131.240:8554/h264ESVideoTest ! rtph264depay ! h264parse ! mp4mux ! filesink location=file.mp4

#==============================================================================================
#pipelinePublic2
#Capture RTSP stream to files splitted to 10 seconds chunks:
gst-launch-1.0 rtspsrc location=rtsp://10.82.131.240:8554/h264ESVideoTest ! rtph264depay ! h264parse ! splitmuxsink location=file%02d.mp4 max-size-time=10000000000 

#==============================================================================================
#pipelinePublic3
#splitmuxsink test (overwritting max 2 files)
gst-launch-1.0 videotestsrc ! x264enc ! h264parse ! splitmuxsink location=file%02d.mp4 max-size-time=3000000000 max-files=2

#==============================================================================================
#pipelinePublic4
# Get TV Channel from DVB and save it in as series of 10sec long MPEG2-TS files
GST_DEBUG_DUMP_DOT_DIR=/tmp gst-launch-1.0 dvbbasebin modulation="QAM 64" trans-mode=8k bandwidth=8 frequency=682000000 code-rate-lp=AUTO code-rate-hp=2/3 guard=4  hierarchy=0 program-numbers=4170 name=src .src ! queue ! tsdemux program-number=4170 name=d  splitmuxsink location=file_%llu.ts max-size-time=1000000000 muxer=mpegtsmux name=mux   d. ! queue ! mpegvideoparse ! queue ! mux.   d. ! queue ! mpegaudioparse ! queue ! mux.audio_%u

#==============================================================================================
#pipelinePublic5
# Create one MPEG2-TS file from series of small chunk MPEG2-TS files
GST_DEBUG_DUMP_DOT_DIR=/tmp gst-launch-1.0 splitmuxsrc location=file*.ts ! filesink location=output.ts

#==============================================================================================
#pipelinePublic6
# Deinterlacing video using transcoding method
gst-launch-1.0 filesrc location=interlaced.ts ! decodebin ! videoconvert ! deinterlace ! videoconvert ! videorate ! "video/x-raw,framerate=25/1" ! avenc_mpeg2video interlaced=false gop-size=25 bitrate=100000000 ! "video/mpeg,profile=main,level=main,parsed=true" ! mpegtsmux ! filesink location=deinterlaced.ts
