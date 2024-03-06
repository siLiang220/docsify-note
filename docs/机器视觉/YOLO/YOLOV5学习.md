## 一、环境安装

### 1.1 安装miniconda

1.  下载地址：[Index of /anaconda/miniconda/ | 清华大学开源软件镜像站 | Tsinghua Open Source Mirror](https://mirrors.tuna.tsinghua.edu.cn/anaconda/miniconda/)
2. 选择下载：`Miniconda3-py38_22.11.1-1-Windows-x86_64.exe`
### 1.2 创建YOLOV5环境
1. 打开Anaconda Prompt 
2. 执行创建环境命令
```sh
conda create -n yolov5 python=3.8
```
3. 激活`yolov5`
```SH
conda activate yolov5
```

4. 查看安装环境的依赖包
```sh
pip list
```

5. 配置pypi国内源
- 清华源：[pypi | 镜像站使用帮助 | 清华大学开源软件镜像站 | Tsinghua Open Source Mirror](https://mirrors.tuna.tsinghua.edu.cn/help/pypi/)
```sh
pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple
```

### 1.3 安装`Pytorch`
[PyTorch官网](https://pytorch.org/)

1. 查看电脑`CUDA`版本 安装小于等于CUDA版本的Pytorch
![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/202401211613347.png)

2. 安装`Pytorch 1.8.2 tls` 版本
```sh
pip3 install torch==1.8.2 torchvision==0.9.2 torchaudio==0.8.2 --extra-index-url https://download.pytorch.org/whl/lts/1.8/cu102
```

### 1.4 安装YoloV5
1. 下载yolov5源码

- [github地址](https://github.com/ultralytics/yolov5)

>[!tip]
>requirements.txt 文件中的torch的版本不是正确的版本，所以需要提前手动安装Pytorch。 

2. 安装依赖
```sh
pip install -r requirements.txt
```

### 1.5 测试代码
1. 项目目录下执行检测代码`detect.py`
```sh
python detect.py
```
2. 下载模型报错需要手动下载到项目目录下重新执行`detect.py`
![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/202401221952642.png)

3. 检测结果
![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/202401222002157.png)

4. 使用电脑显卡跑测试
- 没有在`Anacoda`的环境下执行
- device由默认改为自己电脑显卡的编号，我电脑显卡编号为1（显卡编号在“任务管理器-性能”中查看）
```python
parser.add_argument('--device', default='1', help='cuda device, i.e. 0 or 0,1,2,3 or cpu')
```

## 二、YoloV5参数

- `weights`: 训练好的模型文件
![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/202401222006002.png)
- `source`: 检测的目标，可以是单张图片，文件夹、屏幕摄像头
## 三、基于 torch.hub 的检测方法
1. 安装 `jupyterlab`
```sh
pip install jupyterlab
```
2.  编写`hub_detect.ipynb`
```python
import torch 

# model
model = torch.hub.load("./", "yolov5s", source="local")

# images
img = "./data/images/zidane.jpg"

#result
resutls = model(img)
resutls.show()
```

## 四、数据集构建
### 4.1 数据收集

- 使用`openCV`对视频类型数据抽帧
1. 新建文件夹dataset，将视频放到该目录下
2. 新建`extract.ipynb`
```python
import cv2
import matplotlib.pyplot as plt

video = cv2.VideoCapture("./BVN.mp4")
num = 0         # 计数器
save_step = 30  # 间隔帧
while True:
    ret, frame = video.read()
    if not ret:
        break
    num += 1
    if num % save_step == 0:
        cv2.imwrite("./images/" + str(num) + ".jpg", frame)
```
4.  
### 4.2 标注工具
1.  安装`labelimg`
```sh
pip install labelimg
```
2. cmd命令执行`labelimg`
```sh
labelimg
```

## 五、模型训练

### 5.1 数据调整
- images: 存放图片
	- train: 训练集图片
	- val: 验证集图片
- labels: 存放标签
	- train: 训练集标签文件，要与训练集图片名称一一对应
	- val: 验证集标签文件，要与验证集图片名称一一对应
### 5.2 关键参数
- weight: 预训练的权重文件，基于官方的权重文件训练
- data: 数据集描述文件
![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/202401232122969.png)


1.  复制coco128.yaml 文件重命名bvn.yaml
```yaml
# YOLOv5 🚀 by Ultralytics, AGPL-3.0 license
# COCO128 dataset https://www.kaggle.com/ultralytics/coco128 (first 128 images from COCO train2017) by Ultralytics
# Example usage: python train.py --data coco128.yaml
# parent
# ├── yolov5
# └── datasets
#     └── coco128  ← downloads here (7 MB)

# Train/val/test sets as 1) dir: path/to/imgs, 2) file: path/to/imgs.txt, or 3) list: [path/to/imgs1, path/to/imgs2, ..]
path: ./datasets # dataset root dir
train: images/train # train images (relative to 'path') 128 images
val: images/val # val images (relative to 'path') 128 images
test: # test images (optional)

# Classes
names:
  0: daitu
  1: mingren
  

# Download script/URL (optional)
download: https://ultralytics.com/assets/coco128.zip

```
2. 修改train.py 数据路径
![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/202401232123310.png)

3. 执行`train.py`
>[!tip]
>记住切换vscode python 环境

### 5.3 训练时遇到的问题
![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/202401232129622.png)
### 5.4 训练结果
- best.pt: 最好的训练结果

![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/202401242115645.png)


### 5.5验证训练的模型
```sh
python detect.py --weights runs/train/exp7/weights/best.pt --source datasets/BVN.mp4 --view-img
```