# mjpeg_proxy
Gstreamer mjpeg proxy

Proxying the mjpeg stream from a raspberry pi 1 connected Microsoft Lifecam. Without transcoding for lowest latency.

# raspberry pi (stream to hostip_1)
gst-launch-1.0 -v v4l2src device=/dev/video0  ! "image/jpeg,width=800, height=600,framerate=30/1" ! rtpjpegpay ! udpsink host=hostip_1 port=5003

# linux proxy server (receive stream and tcp serve it)
gst-launch-1.0 -e -v udpsrc port=5003 ! application/x-rtp, encoding-name=JPG, payload=26 ! rtpjpegdepay ! queue ! multipartmux ! tcpserversink port=5010 host=hostip_1

# receiver (connect to tcp server and read stream)
gst-launch-1.0 tcpclientsrc port=5010 host=hostip_1 ! multipartdemux ! jpegdec ! autovideosink
