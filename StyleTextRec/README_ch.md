## Style Text Rec

### 目录
- [工具简介](#工具简介)
- [环境配置](#环境配置)
- [快速上手](#快速上手)
- [高级使用](#高级使用)
- [应用示例](#应用示例)

### 工具简介
<div align="center">
    <img src="doc/images/3.png" width="800">
</div>

Style-Text数据合成工具是基于百度自研的文本编辑算法《Editing Text in the Wild》https://arxiv.org/abs/1908.03047
不同于常用的基于GAN的数据合成工具，Style-Text主要框架包括 ①文本前景风格迁移模块 ②背景抽取模块 ③融合模块。经过这样三步，就可以迅速实现图片文字风格迁移啦。

<div align="center">
    <img src="doc/images/1.png" width="600">
    <img src="doc/images/2.png" width="600">
</div>

此外，在实际铭牌文本识别场景和韩语文本识别场景，验证了该合成工具的有效性。

### 环境配置

1. 参考[快速安装](../doc/doc_ch/installation.md)，安装PaddleOCR。强烈建议您使用python3环境。
2. 进入`style_text_rec`目录，下载模型，并解压：

```bash
cd style_text_rec
wget https://paddleocr.bj.bcebos.com/dygraph_v2.0/style_text/style_text_models.zip
unzip style_text_models.zip
```

如果您将模型保存再其他位置，请在`configs/config.yml`中修改模型文件的地址，修改时需要同时修改这三个配置：

```
bg_generator:
  pretrain: style_text_models/bg_generator
...
text_generator:
  pretrain: style_text_models/text_generator
...
fusion_generator:
  pretrain: style_text_models/fusion_generator
```

### 快速上手

1. 运行tools/synth_image，生成示例图片：

```python
python3 -m tools.synth_image -c configs/config.yml
```

1. 运行后，会生成`fake_busion.jpg`，即为最终结果。
<div align="center">
    <img src="doc/images/4.jpg" width="300">
</div>
除此之外，程序还会生成并保存中间结果：
   * `fake_bg.jpg`：为风格参考图去掉文字后的背景；
   * `fake_text.jpg`：是用提供的字符串，仿照风格参考图中文字的风格，生成在灰色背景上的文字图片。

2. 如果您想尝试其他风格图像和文字的效果，可以添加style_image,text_corpus和language参数：
```python
python3 -m tools.synth_image -c configs/config.yml --style_image examples/style_images/2.jpg --text_corpus PaddleOCR --language en
```
   * 注意：语言选项和语料相对应，目前我们支持英文、简体中文和韩语。

3. 在`tools/synth_image.py`中，我们还提供了一个`batch_synth_images`方法，可以两两组合语料和图片，批量生成一批数据。

### 高级使用

在开始合成数据集前，需要准备一些素材。

首先，需要风格图片作为合成图片的参考依据，这些数据可以是用作训练OCR识别模型的数据集。本例中使用带有标注文件的数据集作为风格图片.

1. 在`configs/dataset_config.yml`中配置输入数据路径。
   * `StyleSampler`：
     * `method`：使用的风格图片采样方法；
     * `image_home`：风格图片目录；
     * `label_file`：风格图片路径列表文件，如果所用数据集有label，则label_file为label文件路径；
     * `with_label`：标志`label_file`是否为label文件。
   * `CorpusGenerator`：
     * `method`：语料生成方法，目前有`FileCorpus`和`EnNumCorpus`可选。如果使用`EnNumCorpus`，则不需要填写其他配置，否则需要修改`corpus_file`和`language`；
     * `language`：语料的语种；
     * `corpus_file`: 语料文件路径。
   
   我们提供了一批[样例图](https://paddleocr.bj.bcebos.com/dygraph_v2.0/style_text/chkoen_5w.tar)供您试用，下面给出了一些示例：
<div align="center">
    <img src="doc/images/5.png" width="800">
</div>
2. 运行`tools/synth_dataset`合成数据：

   ``` bash
   python -m tools.synth_dataset -c configs/dataset_config.yml
   ```

3. 如果您想使用并行方式来快速合成数据，可以通过启动多个进程，在启动时需要指定不同的`tag`（`-t`），如下所示:

   ```bash
   python3 -m tools.synth_dataset -t 0 -c configs/dataset_config.yml
   python3 -m tools.synth_dataset -t 1 -c configs/dataset_config.yml
   ```


### 应用示例

在完成上述操作后，即可得到用于OCR识别的合成数据集，下面给出了一些数据集生成的示例：
<div align="center">
    <img src="doc/images/6.png" width="800">
</div>
请您参考[OCR识别文档](https://github.com/PaddlePaddle/PaddleOCR/blob/dygraph/doc/doc_ch/recognition.md#%E5%90%AF%E5%8A%A8%E8%AE%AD%E7%BB%83)，完成训练。

下面展示了一些使用合成数据训练的效果：

| 场景     | 字符       | 原始数据 | 测试数据 | 只使用原始数据的识别准确率 | 新增合成数据 | 使用合成数据识别准确率 | 指标提升 |
| -------- | ---------- | -------- | -------- | -------------------------- | ------------ | ---------------------- | -------- |
| 金属表面 | 英文和数字 | 2203     | 650      | 0.5938                     | 20000        | 0.7546                 | 16%      |
| 随机背景 | 韩语       | 5631     | 1230     | 0.3012                     | 100000       | 0.5057                 | 20%      |

### 项目结构
```
style_text_rec
|-- arch
|   |-- base_module.py
|   |-- decoder.py
|   |-- encoder.py
|   |-- spectral_norm.py
|   `-- style_text_rec.py
|-- configs
|   |-- config.yml
|   `-- dataset_config.yml
|-- engine
|   |-- corpus_generators.py
|   |-- predictors.py
|   |-- style_samplers.py
|   |-- synthesisers.py
|   |-- text_drawers.py
|   `-- writers.py
|-- examples
|   |-- corpus
|   |   `-- example.txt
|   |-- image_list.txt
|   `-- style_images
|       |-- 1.jpg
|       `-- 2.jpg
|-- fonts
|   |-- ch_standard.ttf
|   |-- en_standard.ttf
|   `-- ko_standard.ttf
|-- tools
|   |-- __init__.py
|   |-- synth_dataset.py
|   `-- synth_image.py
`-- utils
    |-- config.py
    |-- load_params.py
    |-- logging.py
    |-- math_functions.py
    `-- sys_funcs.py
```