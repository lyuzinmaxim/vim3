There are some CLI commands for Khadas VIM3 Pro board video capturing (MIPI-CSI Khadas cam)

# Setup and check

Switch off physical IR-filter (**in framebuffer mode CTRL+ALT+F1 only**) 
```
v4l2_test –c 1 –p 0 –F 0 –f 0 –D 0 –R 1 –r 2 –d 2 –N 1000 –n 800 –x 0 –w 0 –e 1 –b /dev/fb0 -v /dev/video0  
```

Check input camera formats
```
gst-device-monitor-1.0
```
or
```
v4l2-ctl -d /dev/video0 --list-formats
```

# Saving input raw video

In format RGB24
```
gst-launch-1.0 v4l2src name=vsrc device=/dev/video0 num-buffers=100 ! video/x-raw,width=1920,height=1080,framerate=60/1,format=RGB ! filesink location=/tmp/test.rgb
```
And play it
```
ffplay -f rawvideo -pixel_format rgb24 -video_size 1920x1080 /tmp/test.rgb

```

<details><summary>In other formats:</summary>

   - In format YUY2
      ```
      gst-launch-1.0 v4l2src name=vsrc device=/dev/video0 num-buffers=100 ! video/x-raw,width=1920,height=1080,framerate=60/1,format=YUY2 ! filesink location=.//test.yuy2
      ```
      And play it
      ```
      ffplay -f rawvideo -pixel_format yuyv422 -video_size 1920x1080 test.yuy2

      ```
   - In format UYVY
    ```
    gst-launch-1.0 v4l2src name=vsrc device=/dev/video0 num-buffers=100 ! video/x-raw,width=1920,height=1080,framerate=60/1,format=UYVY ! filesink location=.//test.uyvy
    ```
    And play it
    ```
    ffplay -f rawvideo -pixel_format uyvy422 -video_size 1920x1080 test.uyvy
    ```
  - In format GRAY8
    ```
    gst-launch-1.0 v4l2src name=vsrc device=/dev/video0 num-buffers=100 ! video/x-raw,width=1920,height=1080,framerate=30/1,format=GRAY8 ! filesink location=.//test.gr
    ```
    And play it
    ```
    ffplay -f rawvideo -pixel_format gray -video_size 1920x1080 test.gr

    ```
</details>

Using gstreamer
```
gst-launch-1.0 -v v4l2src device=/dev/video0 ! video/x-raw,framerate=30/1,width=1280,height=720 ! filesink location=/tmp/test.avi sync=false
```

