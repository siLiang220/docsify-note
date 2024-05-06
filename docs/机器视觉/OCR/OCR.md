
## CUDA介绍
https://gitcode.csdn.net/65eec6fc1a836825ed79d292.html?dp_token=eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpZCI6NjE5NjIwLCJleHAiOjE3MTE0MzU1NjUsImlhdCI6MTcxMDgzMDc2NSwidXNlcm5hbWUiOiJ3b25kZXJsaWFuZzk5NiJ9.HwFJ1VX4XhxwULLZtcZKbPMt0vF7iiRWdUsO8qMcxmM

## 百度飞桨OCR

- [文字识别模型套件PaddleOCR](https://www.paddlepaddle.org.cn/tutorials/projectdetail/3977289)
- [2.6 快速开始](https://gitee.com/paddlepaddle/PaddleOCR/blob/release/2.6/doc/doc_ch/quickstart.md)
- [cuda安装](https://aistudio.baidu.com/projectdetail/696822)
- [cudnn下载地址](https://developer.nvidia.com/rdp/cudnn-archive)
- [百度飞桨(PaddlePaddle) - PaddleHub OCR 文字识别简单使用 - VipSoft - 博客园 (cnblogs.com](https://www.cnblogs.com/vipsoft/p/17384874.html))
- [通过paddlehub简单几行代码实现OCR识别_paddlehub 图片识别-CSDN博客](https://blog.csdn.net/qq_42924144/article/details/135134614)

- [paddlehub进行模型训练](https://aistudio.baidu.com/projectdetail/4166171)
- [paddlehub使用](https://www.paddlepaddle.org.cn/tutorials/projectdetail/3949106)
- [通过OCR实现验证码识别-使用文档-PaddlePaddle深度学习平台](https://www.paddlepaddle.org.cn/documentation/docs/zh/practices/cv/image_ocr.html#ocr)
- [PaddleOCR问题汇总-模型微调](https://blog.csdn.net/liubing8609/article/details/121198294)
https://blog.csdn.net/qq_52852432/article/details/131817619


[百度飞桨(PaddlePaddle) - PaddleHub OCR 文字识别简单使用 - VipSoft - 博客园 (cnblogs.com)](https://www.cnblogs.com/vipsoft/p/17384874.html)

[Jimmy/Paddle - 码云 - 开源中国 (gitee.com)](https://gitee.com/VipSoft/Paddle)


[解决Could not locate zlibwapi.dll. Please make sure it is in your library path!] (https://blog.csdn.net/qq_40280673/article/details/132229908)


[PaddleOCR训练属于自己的模型详细教程（从打标，制作数据集，训练到应用，以行驶证识别为例）-CSDN博客](https://blog.csdn.net/qq_52852432/article/details/131817619)

https://ai.baidu.com/ai-doc/AISTUDIO/Clkg7m4y9
## 一、安装CUDA 和CUDNN

### 1.1 查看自己的CUDA device version 
在命令行中输入
```sh
nidia-smi
```

![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/20240321093146.png)

### 1.2 判断下载版本
常说的安装对应版本的CUDA是指的CUDA runtime version，可以理解为Cuda tookit的版本

- 显卡的算力必须与Cuda runtine version 相匹配[显卡算力和与之相匹配的Cuda runtime version](https://en.wikipedia.org/wiki/CUDA#cite_note-38)
- Cuda runtime version必须小于等于Cuda driver version，我的例子中，就是必须小于等于12.1

### 1.3 下载paddlepaddle对应CUDA 和 cudnn

1. 飞桨官网查看支持的CUDA版本
	- [PP飞桨支持的CUDA版本](https://www.paddlepaddle.org.cn/install/old?docurl=/documentation/docs/zh/install/conda/windows-conda.html) 
2. 本次使用的是`paddlepaddle` 2.5 版本下载对应的CUDA 10.2，cuDNN 7.6.5
3. 下载CUDA Toolkit
	- [CUDA Toolkit Archive | NVIDIA Developer](https://developer.nvidia.com/cuda-toolkit-archive)
	- 下载后选择自定义默认安装即可
4. 下载cuDNN7.6.5
	-  登录[Log in | NVIDIA Developer](https://developer.nvidia.com/login)
	- 账号：1782705551@qq.com 密码 Na5749090219.
	- 将cudnn解压后的bin、lib、include 目录下的文件分别复制到cuda 的bin、lib、include目录下

## 二、安装paddlepaddle
### 2.1选用Conda 的安装方式(前提已安装好anacoda)
- 使用CPU
```shell
```
- 使用GPU
```sh
conda install paddlepaddle-gpu==2.5.2 cudatoolkit=10.2 --channel https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/Paddle/
```

### 2.2 安装 PaddleOCR whl包

```pip
pip install "paddleocr>=2.0.1" # 推荐使用2.0.1+版本
```

### 2.3 Python使用

- 参考gitee 文档 [doc/doc_ch/quickstart.md · PaddlePaddle/PaddleOCR - Gitee.com](https://gitee.com/paddlepaddle/PaddleOCR/blob/release/2.5/doc/doc_ch/quickstart.md)

- PaddleOCR提供了一系列测试图片，点击[这里](https://gitee.com/link?target=https%3A%2F%2Fpaddleocr.bj.bcebos.com%2Fdygraph_v2.1%2Fppocr_img.zip)下载并解压，然后在终端中切换到相应目录

- 通过Python脚本使用PaddleOCR whl包，whl包会自动下载ppocr轻量级模型作为默认模型。

```python
from paddleocr import PaddleOCR, draw_ocr
# Paddleocr目前支持的多语言语种可以通过修改lang参数进行切换
# 例如`ch`, `en`, `fr`, `german`, `korean`, `japan`
ocr = PaddleOCR(use_angle_cls=True, lang="ch")  # need to run only once to download and load model into memory
img_path = './imgs/11.jpg'
result = ocr.ocr(img_path, cls=True)
for line in result:
    print(line)
# 显示结果
from PIL import Image
image = Image.open(img_path).convert('RGB')
boxes = [line[0] for line in result]
txts = [line[1][0] for line in result]
scores = [line[1][1] for line in result]
im_show = draw_ocr(image, boxes, txts, scores, font_path='./fonts/simfang.ttf')
im_show = Image.fromarray(im_show)
im_show.save('result.jpg')
```

### 三、模型下载

官方的模型下载地址[PP-OCR系列模型列表](https://gitee.com/paddlepaddle/PaddleOCR/blob/release/2.5/doc/doc_ch/models_list.md#)


### 四  模型训练
#### 遇到中英文识别模型不支持的字符，该如何对模型做微调？[](https://openi.pcl.ac.cn/CYing/PaddleOCR/src/branch/dygraph/doc/doc_ch/FAQ.md#user-content-q-%E9%81%87%E5%88%B0%E4%B8%AD%E8%8B%B1%E6%96%87%E8%AF%86%E5%88%AB%E6%A8%A1%E5%9E%8B%E4%B8%8D%E6%94%AF%E6%8C%81%E7%9A%84%E5%AD%97%E7%AC%A6-%E8%AF%A5%E5%A6%82%E4%BD%95%E5%AF%B9%E6%A8%A1%E5%9E%8B%E5%81%9A%E5%BE%AE%E8%B0%83)

**A**：如果希望识别中英文识别模型中不支持的字符，需要更新识别的字典，并完成微调过程。比如说如果希望模型能够进一步识别罗马数字，可以按照以下步骤完成模型微调过程。

1. 准备中英文识别数据以及罗马数字的识别数据，用于训练，同时保证罗马数字和中英文识别数字的效果；
2. 修改默认的字典文件，在后面添加罗马数字的字符；
3. 下载PaddleOCR提供的预训练模型，配置预训练模型和数据的路径，开始训练。

- 具体实现步骤

1. 定位字典文件：在您的PaddleOCR安装目录或源码目录中，找到ppocr/utils/ppocr_keys_v1.txt文件。例如，如果PaddleOCR安装在/path/to/PaddleOCR目录下，字典文件路径可能是/path/to/PaddleOCR/ppocr/utils/ppocr_keys_v1.txt。
2. 更新字典：打开ppocr_keys_v1.txt文件，确保在文件末尾添加罗马数字字符。罗马数字通常包括以下字符（按需添加）：