
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
- [模型训练]https://blog.csdn.net/qq_52852432/article/details/131817619


[百度飞桨(PaddlePaddle) - PaddleHub OCR 文字识别简单使用 - VipSoft - 博客园 (cnblogs.com)](https://www.cnblogs.com/vipsoft/p/17384874.html)

[Jimmy/Paddle - 码云 - 开源中国 (gitee.com)](https://gitee.com/VipSoft/Paddle)


[解决Could not locate zlibwapi.dll. Please make sure it is in your library path!] (https://blog.csdn.net/qq_40280673/article/details/132229908)


[PaddleOCR训练属于自己的模型详细教程（从打标，制作数据集，训练到应用，以行驶证识别为例）-CSDN博客](https://blog.csdn.net/qq_52852432/article/details/131817619)

https://ai.baidu.com/ai-doc/AISTUDIO/Clkg7m4y9
## 一、安装CUDA 和CUDNN
- [# CUDA｜Windows 系统 CUDA、NVCC、CUDNN 版本查看方法](https://dataartist.blog.csdn.net/article/details/128604798)

### 1.1 查看自己的CUDA device version 

在命令行中输入
```sh
nidia-smi
```

![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/20240321093146.png)

### 1.2 下载CUDA Toolkit
常说的安装对应版本的CUDA是指的CUDA runtime version，可以理解为Cuda tookit的版本

- 显卡的算力必须与Cuda runtine version 相匹配[显卡算力和与之相匹配的Cuda runtime version](https://en.wikipedia.org/wiki/CUDA#cite_note-38)
- Cuda runtime version必须小于等于Cuda driver version，我的例子中，就是必须小于等于12.1

![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/20240509111940.png)

验证安装版本
```bash
nvcc --version
```
### 1.3 下载paddlepaddle对应CUDA 和 cudnn

PaddlePaddle 环境要求是 CUDA Runtime Version
在文档中，描述安装 GPU 版 PaddlePaddle 的环境要求如下：

CUDA 工具包 10.2 配合 cuDNN v7.6.5，如需使用 PaddleTensorRT 推理，需配合 TensorRT7.0.0.11
CUDA 工具包 11.2 配合 cuDNN v8.2.1，如需使用 PaddleTensorRT 推理，需配合 TensorRT8.2.4.2
CUDA 工具包 11.6 配合 cuDNN v8.4.0，如需使用 PaddleTensorRT 推理，需配合 TensorRT8.4.0.6
CUDA 工具包 11.7 配合 cuDNN v8.4.1，如需使用 PaddleTensorRT 推理，需配合 TensorRT8.4.2.4
GPU 运算能力超过 3.5 的硬件设备
但是需要注意，在上述要求中，要求的是 CUDA Runtime Version，而不是 CUDA Driver Version。


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
	![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/20240510090653.png)


## 二 安装PPOCR环境
### 2.1选用Conda 的安装方式(前提已安装好anacoda)
- 使用CPU
```shell
conda install paddlepaddle==2.5.2 --channel https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/Paddle/
```
- 使用GPU
```sh
conda install paddlepaddle-gpu==2.5.2 cudatoolkit=10.2 --channel https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/Paddle/
```


### 2.2 安装 PaddleOCR whl包

```pip
pip install "paddleocr>=2.0.1" # 推荐使用2.0.1+版本
```

### 2.3 测试检测结果

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



官方的模型下载地址[PP-OCR系列模型列表](https://gitee.com/paddlepaddle/PaddleOCR/blob/release/2.5/doc/doc_ch/models_list.md#)


## 三  模型训练
#### 遇到中英文识别模型不支持的字符，该如何对模型做微调？[](https://openi.pcl.ac.cn/CYing/PaddleOCR/src/branch/dygraph/doc/doc_ch/FAQ.md#user-content-q-%E9%81%87%E5%88%B0%E4%B8%AD%E8%8B%B1%E6%96%87%E8%AF%86%E5%88%AB%E6%A8%A1%E5%9E%8B%E4%B8%8D%E6%94%AF%E6%8C%81%E7%9A%84%E5%AD%97%E7%AC%A6-%E8%AF%A5%E5%A6%82%E4%BD%95%E5%AF%B9%E6%A8%A1%E5%9E%8B%E5%81%9A%E5%BE%AE%E8%B0%83)

**A**：如果希望识别中英文识别模型中不支持的字符，需要更新识别的字典，并完成微调过程。比如说如果希望模型能够进一步识别罗马数字，可以按照以下步骤完成模型微调过程。

1. 准备中英文识别数据以及罗马数字的识别数据，用于训练，同时保证罗马数字和中英文识别数字的效果；
2. 修改默认的字典文件，在后面添加罗马数字的字符；
3. 下载PaddleOCR提供的预训练模型，配置预训练模型和数据的路径，开始训练。

- 具体实现步骤

1. 定位字典文件：在您的PaddleOCR安装目录或源码目录中，找到ppocr/utils/ppocr_keys_v1.txt文件。例如，如果PaddleOCR安装在/path/to/PaddleOCR目录下，字典文件路径可能是/path/to/PaddleOCR/ppocr/utils/ppocr_keys_v1.txt。
2. 更新字典：打开ppocr_keys_v1.txt文件，确保在文件末尾添加罗马数字字符。罗马数字通常包括以下字符（按需添加）：



### 创建虚拟环境

输入以下代码用来创建虚拟环境
```bash
conda create -n paddleenv2 python=3.8
```

输入以下指令安装paddlepaddle GPU版本

```
# CUDA 10.2
python3 -m pip install paddlepaddle-gpu==2.5.2 -i https://pypi.tuna.tsinghua.edu.cn/simple
```

> 使用pip paddlepaddle GPU安装报错提示cuda版本错误尚未解决

 克隆PaddleOCR代码仓库首先，确保的系统中安装了Git。然后，打开命令行工具，执行以下命令以克隆PaddleOCR的代码仓库：
```bash
git clone https://github.com/PaddlePaddle/PaddleOCR.git
```

 安装依赖 
```bash
pip install -r requirements.txt
```

###  验证环境

#### 1.验证paddle环境
- 执行以下代码
```python
if __name__ == "__main__":
    import paddle
    paddle.utils.run_check()
```
- 正常检测结果
![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/20240510091817.png)

#### 2.下载所需模型:

- 访问官方[PP-OCR系列模型列表](https://gitee.com/paddlepaddle/PaddleOCR/blob/release/2.5/doc/doc_ch/models_list.md#)获取官方模型

- 需要下载文本检测模型和文本识别模型，选择下载推理模型。参考下图：
 
![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/20240510090839.png)

![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/20240510090930.png)

#### 3. 解压模型
- 在项目根目录创建`inference_model`文件夹
- 将下载的推理模型压缩包解压到`inference_model`文件夹。

![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/20240510091045.png)


在项目根目录下使用终端执行指令，其中`image_dir`为所要识别的图片路径，`det_model_dir`为下载的文字检测模型，`rec_model_dir`为下载的文字识别模型。

```bash
python tools/infer/predict_system.py  --image_dir="C:\Users\User\Desktop\test.jpg" --det_model_dir="./inference_model/ch_PP-OCRv3_det_infer/" --rec_model_dir="./inference_model/ch_PP-OCRv3_rec_infer"
```

#### 4. 可能遇到的错误
错误处理1：`（OMP: Error #15: Initializing libiomp5md.dll, but found libiomp5 already initialized）`执行以下命令或者删除一个 `libiomp5md.dll`
```bash
set KMP_DUPLICATE_LIB_OK=True
```

错误处理2:提示numpy.int 在NumPy 1.20中已经弃用，而在NumPy 1.24中删除。解决办法：指定安装1.22版本。
![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/20240510091343.png)
- 卸载numpy
```bash
pip uninstall numpy
```
- 安装 1.22.0 版本
```bash
 pip install numpy==1.22.0
```

#### 5. 识别结果如图所示说明成功
![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/20240510093423.png)

### 标注图片

### 1. 安装PPOCRLabel

- [参考官方安装文档](https://gitee.com/paddlepaddle/PaddleOCR/blob/dygraph/PPOCRLabel/README_ch.md) 
- 方法一：进入项目根目录下再cd 到 `PPOCRLabel` 执行以下代码打开打标软件。(但是需要下载很多依赖)
``` python
python PPOCRLabel.py --lang ch
```
- 方法二：使用pip安装 PPOCRLabel
```bash
pip install PPOCRLabel # 安装
```

- 启动运行
```bash
PPOCRLabel --lang ch  # 启动【普通模式】，用于打【检测+识别】场景的标签
```
 - 启动遇到问题
 ![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/20240510160905.png)
解决方法
```python
pip install opencv-python install "opencv-python-headless<4.3"
```
### 2. 标注图片
![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/20240510165730.png)

然后从第一张开始检查，漏打标的按下Q框出字体，打标文字错误的，点击方框，在右边修改

全部打标完成之后，点击文件选择导出标记结果，再点击文件选择导出识别结果，完成后在文件夹多出四个文件fileState，Label，rec_gt, crop_img。其中crop_img中的图片用来训练文字识别模型，fileState记录图片的打标完成与否，Label为训练文字检测模型的标签，rec_gt为训练文字识别模型的标签。

### 2. 修改字典文件
项目中找到并更新`ppocr/utils/ppocr_keys_v1.txt`文件，加入罗马数字字符。

4. 准备训练数据

5. 配置训练参数
6. 下载预训练模型
7. 开始训练
```bash
python tools/train.py -c configs/your_new_config.yml -o Global.pretrained_model=/path/to/your/pretrained/model -o Global.save_interval=1000
```
8. 导出模型



https://blog.csdn.net/yinqinggong/article/details/134823777?utm_medium=distribute.pc_relevant.none-task-blog-2~default~baidujs_baidulandingword~default-0-134823777-blog-134715664.235^v43^pc_blog_bottom_relevance_base5&spm=1001.2101.3001.4242.1&utm_relevant_index=3