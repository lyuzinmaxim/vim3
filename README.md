# vim3
Notes about deployment neural nets on Khadas VIM3 

Download last SDK version 
```
git clone https://gitlab.com/khadas/aml_npu_sdk.git
```


Download **.cfg**-file
```
wget --load-cookies /tmp/cookies.txt "https://docs.google.com/uc?export=download&confirm=$(wget --quiet --save-cookies /tmp/cookies.txt --keep-session-cookies --no-check-certificate 'https://docs.google.com/uc?export=download&id=1FsZK5DBghDMqf1u75r_gNdMlpouUgns2' -O- | sed -rn 's/.*confirm=([0-9A-Za-z_]+).*/\1\n/p')&id=1FsZK5DBghDMqf1u75r_gNdMlpouUgns2" -O yolov3.cfg && rm -rf /tmp/cookies.txt 
```

Download **.weights**-file
```
wget --load-cookies /tmp/cookies.txt "https://docs.google.com/uc?export=download&confirm=$(wget --quiet --save-cookies /tmp/cookies.txt --keep-session-cookies --no-check-certificate 'https://docs.google.com/uc?export=download&id=1L94CcYcRD0tyju0PFxBRFfY4rBdJsYt1' -O- | sed -rn 's/.*confirm=([0-9A-Za-z_]+).*/\1\n/p')&id=1L94CcYcRD0tyju0PFxBRFfY4rBdJsYt1" -O yolov3_24_feb_best.weights && rm -rf /tmp/cookies.txt 
```

Or use officially (pjreddie)
```
wget https://github.com/yan-wyb/darknet/blob/master/cfg/yolov3.cfg
wget https://raw.githubusercontent.com/yan-wyb/darknet/master/cfg/yolov3.cfg
```
416x416 image
```
wget https://redcanoebrands.com/wp-content/uploads/2018/02/Womens-CBC74-WB-tshirt-416x416.jpg
```
608x608 image
```
wget https://v2.cimg.co/authors/21/5a65ac2ee2b6e.jpg
```

```
mkdir ~/model && cp ~/yolov3.cfg ~/model/ && cp ~/yolov3_24_feb_best.weights ~/model/
```

There are two opportunities:
1. Use official prebuilt docker image with older version (5.7.0) - maybe it caused an error when generating case code (https://forum.khadas.com/t/npu-sdk-6-4-4-3-release/12273/37
)
```
docker pull numbqq/npu-sdk:latest
```
```
docker run -it --name npu-sdk -v <model path>:/model --privileged  --cap-add SYS_ADMIN numbqq/npu-sdk
```

2. Build own image with latest version (6.4.4.3) (not tried)

Edit *requirements.txt* in ```~/aml_npu_sdk_6.4.4.3/acuity-toolkit``` 

tensorflow=2.0.0 -> tensorflow=2.0.0a0

```
cd aml_npu_sdk_6.4.4.3 && docker build -t khadas .
```
```
docker run -it --name khadas -v ~/model:/model --privileged  --cap-add SYS_ADMIN khadas
```

Some DOCKER tips:
(according to https://medium.com/codingthesmartway-com-blog/docker-beginners-guide-part-1-images-containers-6f3507fffc98)

1. Show stopped but existing images 
```
docker ps -a 
```
2. Remove images
```
docker rm {}
```
3. Start stopped container (and after **cd** command in container)
```
docker start {container name} -i
```

# SDK workflow:

Clone actual repo
```
git clone https://gitlab.com/khadas/aml_npu_sdk.git
```

Make an intermediate file with *.json* and *.data* format

```
cd /acuity-toolkit && ./bin/convertdarknet --net-input /model/yolov3.cfg \
--weight-input /model/yolov3.weights \
--data-output yolov3.data \
--net-output yolov3.json
```

Download small quantization dataset:
```
pip3 install gdown

cd /acuity-toolkit && gdown https://drive.google.com/uc?id=1310_XNWwHWhJJiMs14URMHkj43dvD-2o
unzip archive_aml2
```
**ALL IMAGES SHOULD HAVE THE SAME RESOLUTION**

Quantizing the model - output files: *.quantized*, *.data* 

```
./bin/tensorzonex --action quantization \
--quantized-dtype asymmetric_affine-u8 \
--channel-mean-value '0 0 0 256' \
--source text \
--source-file /acuity-toolkit/archive/dataset.txt \
--model-input yolov3.json \
--model-data yolov3.data \
--reorder-channel '0 1 2' \
--quantized-rebuild \
--batch-size 2 \
--epochs 50
```
Generating case files for inference
https://hackersandslackers.com/multiple-versions-python-ubuntu/

```
./bin/ovxgenerator --model-input yolov3.json \
--data-input yolov3.data \
--channel-mean-value '0 0 0 256' \
--reorder-channel '0 1 2' \
--export-dtype quantized \
--model-quantize yolov3.quantize \
--optimize VIPNANOQI_PID0X88 \
--viv-sdk ../bin/vcmdtools \
--pack-nbg-unify
```

Infere on ?image?

```
./bin/tensorzonex \
--action inference \
--source text \
--source-file /acuity-toolkit/archive/test.txt \
--channel-mean-value '0 0 0 256' \
--model-input yolov3.json \
--model-data yolov3.data \
--dtype quantized
```

Download from google drive
```
gdown https://drive.google.com/uc?id=
```

Remove write-protected files
```
rm -f /path/to/file
```

Compiling original 80-classes yolo in aml_npu_app:
```
COMPILE /home/khadas/aml_npu_app/DDK_6.3.3.4/detect_library/model_code/detect_yolo_v3/yolov3_process.c
  COMPILE /home/khadas/aml_npu_app/DDK_6.3.3.4/detect_library/model_code/detect_yolo_v3/vnn_yolov3.c
vnn_yolov3.c: In function ???vnn_CreateYolov3???:
vnn_yolov3.c:145:29: warning: unused variable ???data??? [-Wunused-variable]
  145 |     uint8_t *               data;
      |                             ^~~~
At top level:
vnn_yolov3.c:94:17: warning: ???load_data??? defined but not used [-Wunused-function]
   94 | static uint8_t* load_data
      |                 ^~~~~~~~~
  COMPILE /home/khadas/aml_npu_app/DDK_6.3.3.4/detect_library/model_code/detect_yolo_v3/yolo_v3.c
make: Nothing to be done for 'all'.
```
Compiling custom 1-class yolo in aml_npu_app:
```
  COMPILE /home/khadas/aml_npu_app/DDK_6.3.3.4/detect_library/model_code/detect_yolo_v3/yolov3_process.c
  COMPILE /home/khadas/aml_npu_app/DDK_6.3.3.4/detect_library/model_code/detect_yolo_v3/vnn_yolov3.c
vnn_yolov3.c: In function ???vnn_CreateYolov3???:
vnn_yolov3.c:145:29: warning: unused variable ???data??? [-Wunused-variable]
  145 |     uint8_t *               data;
      |                             ^~~~
At top level:
vnn_yolov3.c:94:17: warning: ???load_data??? defined but not used [-Wunused-function]
   94 | static uint8_t* load_data
      |                 ^~~~~~~~~
make: Nothing to be done for 'all'.

```
Compiling custom 1-class yolov4 in aml_npu_app:
```
  COMPILE /home/khadas/aml_npu_app/DDK_6.3.3.4/detect_library/model_code/detect_yolo_v4/yolov4_process.c
  COMPILE /home/khadas/aml_npu_app/DDK_6.3.3.4/detect_library/model_code/detect_yolo_v4/vnn_yolov4.c
vnn_yolov4.c: In function ???vnn_CreateYolov4???:
vnn_yolov4.c:145:29: warning: unused variable ???data??? [-Wunused-variable]
  145 |     uint8_t *               data;
      |                             ^~~~
At top level:
vnn_yolov4.c:94:17: warning: ???load_data??? defined but not used [-Wunused-function]
   94 | static uint8_t* load_data
      |                 ^~~~~~~~~
make: Nothing to be done for 'all'.
khadas@Khadas:~/aml_npu_app/detect_
```



Extra materials:
1. Here it's wrote, that should be used new version of SDK
#https://forum.khadas.com/t/npu-sdk-6-4-4-3-release/12273/37
2. sudo apt install libjpeg9-dev
3. 


DSI display
