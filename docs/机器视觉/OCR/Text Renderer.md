
## 一、准备工作

### 基于环境

使用minicoda 创建python3.7运行环境，项目中`dockerfile`使用的python 环境是3.7

```bash
conda create -n renderer2 python=3.7
```
###  下载源码
- [github源代码下载](https://github.com/oh-my-ocr/text_renderer)

- 在项目的根目录下看到一个`example_data`文件夹
![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/20240521090045.png)

以下对`example_data`文件夹解释
1. `bg文件夹`，存放要产生的图片的背景（图片颜色不重要，因为产出的图都是灰度图）
2. `char文件夹` 默认有两个txt文件，`chn.txt`和`eng.txt`，里面放的就是要产生的 文本的字符，一行一个
3. `font文件夹` 存放你要放在背景图上的文字的字体，也就是产生特定字体的文字图片，我这里主要是晶体管/七段数码管的字体（默认只有一个 `simsun.ttf` 宋体）
4. `font_list文件夹` 默认有一个`font_list.txt`文件，每行一个字体名称（对应于font文件夹中的.ttf文件）
5. `output`文件夹 是存放产出的图片及标签的文件， 每次生成文件是追加不会覆盖
  6. `text文件夹` 默认有三个txt文件，分别是`chn_text.txt`，`eng_text.txt`，`enum_text.txt`
	- `chn_text.txt`里存放着中文文章（有标点符号，分段的那种），程序会自动从里面随机截取一段生成图片；
	- `eng_text.txt`里也是一段英文文档（有空格 有标点符号 分段的那种）
	- `enum_text.txt`里每行一串字符，会随机从这个文件中的行里挑选一行，然后结合背景，产生图片
### 安装

进入项目根目录执行以下命令
```bash
pip3 install -r docker/requirements.txt
```

## 代码修改
- example.py中，只需要保留与enum_data相关的部分，其余都注释掉就可以
-  修改产生图片的数量 `num_image`



##

https://blog.csdn.net/Castlehe/article/details/115489196