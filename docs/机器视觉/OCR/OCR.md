
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


### 数据合成

[数据合成](https://github.com/PaddlePaddle/PaddleOCR/blob/static/doc/doc_ch/data_synthesis.md)
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
### 3. 标注图片
![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/20240510165730.png)

然后从第一张开始检查，漏打标的按下Q框出字体，打标文字错误的，点击方框，在右边修改

全部打标完成之后，点击文件选择导出标记结果，再点击文件选择导出识别结果，完成后在文件夹多出四个文件fileState，Label，rec_gt, crop_img。其中crop_img中的图片用来训练文字识别模型，fileState记录图片的打标完成与否，Label为训练文字检测模型的标签，rec_gt为训练文字识别模型的标签。

![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/1715332794653.png)
在PaddleOCR项目根目录下建立`train_data`文件夹，并且将打标签生成的文件和图片放在该文件夹下。
![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/20240510172236.png)


### 4. 划分数据集
打开终端进入`PPOCRLabel`的文件夹下，执行以下代码进行数据集的划分
```python
python gen_ocr_train_val_test.py --trainValTestRatio 6:2:2 --datasetRootPath ../train_data/roman
```

输入指令后，在train_data文件夹下会出现以下文件，其中det是用来训练文字检测的数据集，rec是用来训练文字识别的数据集。此时可以删去roman。

![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/bb6f02dcee27ec5f12512120f3141c1.png)

![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/39ae8b701586eea47f7f44b41678b75.png)

### 5. 修改字典文件(修改字典尚未实现)

- [数据集说明](https://github.com/PaddlePaddle/PaddleOCR/blob/release/2.6/doc/doc_ch/recognition.md#12-%E8%87%AA%E5%AE%9A%E4%B9%89%E6%95%B0%E6%8D%AE%E9%9B%86)
项目中找到并更新`ppocr/utils/ppocr_keys_v1.txt`文件，加入罗马数字字符。
### 6.  训练文字检测模型
#### 6.1下载预训练模型
- [官方预训练模型下载](https://gitee.com/paddlepaddle/PaddleOCR/blob/release/2.5/doc/doc_ch/models_list.md)
- 下载中文检测模型和中文识别模型，选择下载检测模型，参考下图
![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/20240511093126.png)

![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/20240511093153.png)

下载之后在`PaddleOCR`项目根目录下建立`pretrain_models`文件夹，并将训练模型解压至该文件夹下。如下图所示
![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/20240511093428.png)

#### 6.2模型配置文件

- 复制`ch_PP-OCRv3_det_student.yml`修改为`ch_PP-OCRv3_det_student_roman.yml` 并对以下配置参数进行修改

![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/8840349b7f320b7356315ddfb5e35fd.png)

- 全局配置

```yml
Global:
  debug: false
  use_gpu: true
  epoch_num: 500
  log_smooth_window: 20
  print_batch_step: 10
  save_model_dir: ./output/ch_PP-OCR_V3_det/
  save_epoch_step: 100
  eval_batch_step:
  - 0
  - 400
  cal_metric_during_train: false
  pretrained_model: ./pretrain_models/ch_PP-OCRv3_det_distill_train/student
  checkpoints: null
  save_inference_dir: null
  use_visualdl: false
  infer_img: doc/imgs_en/img_10.jpg
  save_res_path: ./checkpoints/det_db/predicts_db.txt
  distributed: true
```


![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/1126e527813c566932c847399b8ed42.png)

- 训练集位置
![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/fd59d32da58f97cf26fb9130532d712.png)

- 验证集位置

![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/95627653b472dadbb21d0f76c0ffb52.png)

#### 6.3 模型训练
 在项目根目录下使用终端执行以下命令开始模型训练
```bash
python tools/train.py -c configs/det/ch_PP-OCRv3/ch_PP-OCRv3_det_student_roman.yml
```

- 训练输出
![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/5d65408c0eda85fc679e5358e91f0dd.png)

- 这个是训练完成保存的模型
![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/f4a888ed7f8998ea8f857a79f4a5b86.png)

#### 6.4 测试训练模型
在项目下执行以下指令进行测试， 其中`Global.pretrained_model`是我们训练好并且需要测试的模型，`Global.infer_img`为所要检测的图片路径。
```bash
python tools/infer_det.py -c configs/det/ch_PP-OCRv3/ch_PP-OCRv3_det_student_roman.yml -o Global.pretrained_model=output/ch_PP-OCR_V3_det/latest.pdparams Global.infer_img="D:\code\ocr\ppocr_img\imgs\_20240506201155.jpg"
```

执行结果
![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/59beeb7bbc9438dc062d680b5abe376.png)


### 7. 训练文字识别模型

#### 7.1配置模型文件

复制`ch_PP-OCRv3_rec.yml`修改为`OCRv3_rec_distillation_roman.yml` 并对以下配置参数进行修改

![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/38c6dadd0d02893e0aabbc4fc1ea1e9.png)


- 修改预训练模型位置(去掉后缀`.pdparams`)
![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/20240511100922.png)


- 修改训练集
![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/79857d5601c77e9b0c52246b2d98234.png)

- 修改验证集
![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/aba5aef97c1a971fb07d8fa039e10c8.png)

#### 7.2模型训练

 在项目根目录下使用终端执行以下命令开始模型训练
```bash
python tools/train.py -c configs/rec/PP-OCRv3/ch_PP-OCRv3_rec_distillation_roman.yml
```


#### 7.3 模型训练log
```
[2022/02/22 07:58:05] root INFO: epoch: [1/800], iter: 10, lr: 0.000000, loss: 0.754281, acc: 0.000000, norm_edit_dis: 0.000008, reader_cost: 0.55541 s, batch_cost: 0.91654 s, samples: 1408, ips: 153.62133
[2022/02/22 07:58:13] root INFO: epoch: [1/800], iter: 20, lr: 0.000001, loss: 0.924677, acc: 0.000000, norm_edit_dis: 0.000008, reader_cost: 0.00236 s, batch_cost: 0.28528 s, samples: 1280, ips: 448.68599
[2022/02/22 07:58:23] root INFO: epoch: [1/800], iter: 30, lr: 0.000002, loss: 0.967231, acc: 0.000000, norm_edit_dis: 0.000008, reader_cost: 0.14527 s, batch_cost: 0.42714 s, samples: 1280, ips: 299.66507
[2022/02/22 07:58:31] root INFO: epoch: [1/800], iter: 40, lr: 0.000003, loss: 0.895318, acc: 0.000000, norm_edit_dis: 0.000008, reader_cost: 0.00173 s, batch_cost: 0.27719 s, samples: 1280, ips: 461.77252
```

log 中自动打印如下信息：

|      字段       |       含义        |
| :-----------: | :-------------: |
|     epoch     |     当前迭代轮次      |
|     iter      |     当前迭代次数      |
|      lr       |      当前学习率      |
|     loss      |     当前损失函数      |
|      acc      |   当前batch的准确率   |
| norm_edit_dis | 当前 batch 的编辑距离  |
|  reader_cost  | 当前 batch 数据处理耗时 |
|  batch_cost   |  当前 batch 总耗时   |
|    samples    | 当前 batch 内的样本数  |
|      ips      |    每秒处理图片的数量    |
#### 7.3 模型测试

 在终端中输入以下指令进行测试。 其中`Global.pretrained_model`是我们训练好并且需要测试的模型，`Global.infer_img`为所要检测的图片路径。
```bash
python tools/infer_rec.py -c configs/rec/PP-OCRv3/ch_PP-OCRv3_rec_distillation_roman.yml -o Global.pretrained_model=output/rec_ppocr_v3_distillation/latest.pdparams Global.infer_img="D:\code\ocr2\PaddleOCR\train_data\roman\2.jpg"
```

- 识别结果


4. 准备训练数据

5. 配置训练参数
6. 下载预训练模型
7. 开始训练
```bash
python tools/train.py -c configs/your_new_config.yml -o Global.pretrained_model=/path/to/your/pretrained/model -o Global.save_interval=1000
```
8. 导出模型

### 模型微调

[模型微调](https://github.com/PaddlePaddle/PaddleOCR/blob/release/2.6/doc/doc_ch/finetune.md)


https://blog.csdn.net/yinqinggong/article/details/134823777?utm_medium=distribute.pc_relevant.none-task-blog-2~default~baidujs_baidulandingword~default-0-134823777-blog-134715664.235^v43^pc_blog_bottom_relevance_base5&spm=1001.2101.3001.4242.1&utm_relevant_index=3

[模型训练](https://aistudio.baidu.com/modelsdetail/17?modelId=17)

[文本识别训练教程](https://github.com/PaddlePaddle/PaddleOCR/blob/release%2F2.6/doc/doc_ch/recognition.md)
[文本识别训练教程](https://gitee.com/PaddlePaddle/PaddleOCR/blob/release%2F2.6/doc/doc_ch/recognition.md)

[模型微调](https://gitee.com/paddlepaddle/PaddleOCR/blob/release/2.6/doc/doc_ch/finetune.md)

## 官方支持
- https://aistudio.baidu.com/community/channel/610