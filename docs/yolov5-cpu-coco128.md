```bash
# 2 skylake core with 8gb ram
# debian 11 (bullseye)

# docker CE 23.0.0, containerd 1.6.9
$ docker run -dit --rm --name test debian:bookworm-20230725-slim

-----------------------------
apt-get update
apt-get install git curl unzip nano python3-pip
# also try: apt-get install libgl1
apt-get install ffmpeg libsm6 libxext6
rm /usr/lib/python3.11/EXTERNALLY-MANAGED

cd /usr/src
mkdir datasets && cd datasets
curl -LO https://ultralytics.com/assets/coco128.zip
unzip coco128.zip
cd ..

git clone https://github.com/ultralytics/yolov5
cd yolov5
sed -i 's/# onnx>=.*/onnx>=1.12.0/g' requirements.txt
pip install -r requirements.txt
mkdir -p /root/.config/Ultralytics
curl -Lo /root/.config/Ultralytics/Arial.ttf https://ultralytics.com/assets/Arial.ttf
curl -LO https://github.com/ultralytics/yolov5/releases/download/v7.0/yolov5s.pt # or yolov5n.pt
nano data/coco128.yaml
# also try remove "--workers 0" for "--shm-size=5gb" or "--ipc=host" in docker run
python3 train.py --device cpu --workers 0 --batch-size 3 --img 640 --epochs 3 --data coco128.yaml --weights yolov5s.pt

# Validate the trained model for Precision, Recall, and mAP
python3 val.py --device cpu --workers 0 --weights yolov5s.pt --data coco128.yaml --img 640
ls runs/val/exp2/

# Run inference using the trained model on your images or videos
python detect.py --weights yolov5s.pt --source path/to/images
# 0=persion, 1=bicycle, 32=sports ball, 38=tennis racket
python3 detect.py --weights yolov5s.pt --source ../datasets/coco128/images/train2017/000000000531.jpg

# Export the trained model to other formats for deployment
# onnx and openvino runs fast on CPU, tflite good for gpu
python3 export.py --weights yolov5s.pt --data coco128.yaml --include onnx
ls yolov5s.onnx


#Comet: run 'pip install comet_ml' to automatically track and visualize YOLOv5 🚀 runs in Comet
#TensorBoard: Start with 'tensorboard --logdir runs/train', view at http://localhost:6006/
--------------------

$ docker cp test:/usr/src/yolov5/yolov5s.onnx ./
$ ls yolov5s.onnx
```
