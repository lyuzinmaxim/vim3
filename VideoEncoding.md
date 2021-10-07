There are some CLI commands for Khadas VIM3 Pro board video capturing (MIPI-CSI Khadas cam)

# Setup and check

Switch off physical IR-filter (in framebuffer mode CTRL+ALT+F1) 
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
