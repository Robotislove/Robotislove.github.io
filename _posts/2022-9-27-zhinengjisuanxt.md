---
layout: post
title: AI嵌入式和智能计算系统总结学习笔记
date: 2022-09-29
author: lau
tags: [AI, Blog]
comments: true
toc: false
pinned: false



---

AI嵌入式和智能计算系统总结学习笔记。

## 概述

![](https://tuchuangs.com/imgs/2022/09/29/601eda7ed25768f9.png)

## 卷积及其矩阵加速

### 卷积操作的定义

卷积神经网络依旧是层级网络，只是层的功能和形式做了变化，可以说是传统神经网络的一个改进。卷积网络在本质上是一种输入到输出的映射，它能够学习大量的输入与输出之间的映射关系，而不需要任何输入和输出之间的精确的数学表达式，只要用已知的模式对卷积网络加以训练，网络就具有输入输出对之间的映射能力。

卷积就是卷积核和另外一个矩阵 (图片) 对应位置相乘然后结果相加。

![](https://www.mobibrw.com/wp-content/uploads/2019/04/conv.gif)

上图展示的是 `stride=1` 的情形, 既每次移动一个像素的位置, 根据需求, 也可以使用其他的 `stride`。

### 卷积计算的加速

对于大的卷积核, 加速方法一般是使用傅里叶变换 (或者其加强版: 快速傅里叶变换), 但是, 对于比较小的卷积核, 其转换到频域的计算量已经大于直接在空域进行卷

积的计算量, 所以, 我们会发现在主流的深度学习框架中, 一般是直接在空域中进行卷积计算, 其加速计算的方法就是把卷积操作转换成矩阵相乘 (因为有很多优化了

的线性代数计算库和 CUDA), 下面这张图充分说明了具体过程 (2 维的情形).

![](https://www.mobibrw.com/wp-content/uploads/2019/04/conv_matrix.png)

- 在上图中, `input features`每一个二维矩阵对应与 RGB 图像的 channel 或者是 feature map 中的 channel
- 目前常见的卷积都是 cross channel 的卷积, 即卷积核矩阵是 3 维的 (width, height, depth), depth 的大小和 feature map 的 depth.(depth 就是有几张 feature map, 不要被高大上的名词迷惑了)
- 三维卷积核的方法与 2 维的类似, 也是与 feature map 中相应的一个 3 维的矩阵对应位置元素相乘然后相加
- 2 维的卷积相当于 depth=1 的 3 维的卷积

### im2col in Python

```c++
import numpy as np
def get_im2col_indices(x_shape, field_height, field_width, padding=1, stride=1):
  # First figure out what the size of the output should be
  N, C, H, W = x_shape
  assert (H + 2 * padding - field_height) % stride == 0
  assert (W + 2 * padding - field_height) % stride == 0
  out_height = (H + 2 * padding - field_height) / stride + 1
  out_width = (W + 2 * padding - field_width) / stride + 1
  i0 = np.repeat(np.arange(field_height), field_width)
  i0 = np.tile(i0, C)
  i1 = stride * np.repeat(np.arange(out_height), out_width)
  j0 = np.tile(np.arange(field_width), field_height * C)
  j1 = stride * np.tile(np.arange(out_width), out_height)
  i = i0.reshape(-1, 1) + i1.reshape(1, -1)
  j = j0.reshape(-1, 1) + j1.reshape(1, -1)
  k = np.repeat(np.arange(C), field_height * field_width).reshape(-1, 1)
  return (k, i, j)
 
 
def im2col_indices(x, field_height, field_width, padding=1, stride=1):
  """ An implementation of im2col based on some fancy indexing """
  # Zero-pad the input
  p = padding
  x_padded = np.pad(x, ((0, 0), (0, 0), (p, p), (p, p)), mode='constant')
  k, i, j = get_im2col_indices(x.shape, field_height, field_width, padding, stride)
  cols = x_padded[:, k, i, j]
  C = x.shape[1]
  cols = cols.transpose(1, 2, 0).reshape(field_height * field_width * C, -1)
  return cols
```

## 智能计算系统



### RK3568加速方案

> 本文介绍了YOLOv5s算法在国产瑞芯微电子RK3399ProD上的部署推理.介绍整个的流程，并基于RK3399Pro简单的介绍下RKNN的Python接口使用，并记录一些踩过的坑。本文仅做交流使用，文中有什么理解的不到位的或者谬误，也欢迎大家能够交流下。

1. 部署流程

  - 在服务器端或者电脑端进行训练，训练完成后将.pt文件转换成ONNX格式

  - 电脑端使用rknn-toolkit1.6.0将ONNX模型转换成RKNN模型

  - RK3399Pro中进行模型推理其中第二步，转换模型，可以在装了rknn-toolkit的rk3399Pro中，之前不知道的时候都是在这个开发板中进行转换的，不过

    还是建议在电脑端Ubuntu18.04系统下进行转换，可以将量化的batch设置的大一些,量化的速度快一些。本文采用的开发板为信迈科技XM-3399-02。

拉取rockchip官方git [https://github.com/rockchip-linux/rknn-toolkit](https://link.zhihu.com/?target=https%3A//github.com/rockchip-linux/rknn-toolkit)，配置环境。

#### 服务器端或者电脑端训练YOLOv5s

项目地址： https://github.com/littledeep/YOLOv5-RK3399Pro

训练这部分并不用多说什么，仓库拉出来，根据自己的数据集改好参数，然后进行训练，YOLOv5的训练还是挺友好的。

不过训练YOLOv5的过程中，我选择在common文件中，将silu层全部换成了RuLe层，因为对准换后的RKNN模型有加速作用，这样总体的mAP会稍微降一些，经过测试总体的mAP降低的。
#### 模型转换--->ONNX

根据github项目需要先将PyTorch训练出的.pt模型转换成ONNX格式。根据项目的Repo直接转换即可。 在命令行输入 ：
`python3 models/export_op.py --rknn_mode`

#### 模型转换--->RKNN

将转换出来的模型xxxx.onnx复制到convert文件夹下面，convert文件夹需要有转换脚本，dataset.txt的量化文件，量化图片。量化的图片建议200张，batch尽量设置的大一些。 在命令行输入：
`python rknn_convert.py`
rknn_convert.py文件代码

```python
import yaml
from rknn.api import RKNN
import cv2
 
_model_load_dict = {
    'caffe': 'load_caffe',
    'tensorflow': 'load_tensorflow',
    'tflite': 'load_tflite',
    'onnx': 'load_onnx',
    'darknet': 'load_darknet',
    'pytorch': 'load_pytorch',
    'mxnet': 'load_mxnet',
    'rknn': 'load_rknn',
    }
 
yaml_file = './config.yaml'
 
 
def main():
    with open(yaml_file, 'r') as F:
        config = yaml.load(F)
    # print('config is:')
    # print(config)
 
    model_type = config['running']['model_type']
    print('model_type is {}'.format(model_type))#检查模型的类型
 
    rknn = RKNN(verbose=True)
 
 
 
#配置文件
    print('--> config model')
    rknn.config(**config['config'])
    print('done')
 
 
    print('--> Loading model')
    load_function = getattr(rknn, _model_load_dict[model_type])
    ret = load_function(**config['parameters'][model_type])
    if ret != 0:
        print('Load yolo failed! Ret = {}'.format(ret))
        exit(ret)
    print('done')
 
    ####
    #print('hybrid_quantization')
    #ret = rknn.hybrid_quantization_step1(dataset=config['build']['dataset'])
 
 
    if model_type != 'rknn':
        print('--> Building model')
        ret = rknn.build(**config['build'])
        print('acc_eval')
        rknn.accuracy_analysis(inputs='./dataset1.txt', target='rk3399pro')
        print('acc_eval done!')
 
        if ret != 0:
            print('Build yolo failed!')
            exit(ret)
    else:
        print('--> skip Building model step, cause the model is already rknn')
 
 
#导出RKNN模型
    if config['running']['export'] is True:
        print('--> Export RKNN model')
        ret = rknn.export_rknn(**config['export_rknn'])
        if ret != 0:
            print('Init runtime environment failed')
            exit(ret)
    else:
        print('--> skip Export model')
 
 
#初始化
    print('--> Init runtime environment')
    ret = rknn.init_runtime(**config['init_runtime'])
    if ret != 0:
        print('Init runtime environment failed')
        exit(ret)
    print('done')
 
 
    print('--> load img')
    img = cv2.imread(config['img']['path'])
    print('img shape is {}'.format(img.shape))
    img = cv2.cvtColor(img, cv2.COLOR_BGR2RGB)
    inputs = [img]
    print(inputs[0][0:10,0,0])
#推理
    if config['running']['inference'] is True:
        print('--> Running model')
        config['inference']['inputs'] = inputs
        #print(config['inference'])
        outputs = rknn.inference(inputs)
        #outputs = rknn.inference(config['inference'])
        print('len of output {}'.format(len(outputs)))
        print('outputs[0] shape is {}'.format(outputs[0].shape))
        print(outputs[0][0][0:2])
    else:
        print('--> skip inference')
#评价
    if config['running']['eval_perf'] is True:
        print('--> Begin evaluate model performance')
        config['inference']['inputs'] = inputs
        perf_results = rknn.eval_perf(inputs=[img])
    else:
        print('--> skip eval_perf')
 
 
if __name__ == '__main__':
    main()
```

```c
running:
  model_type: onnx       # 转换模型的类型
  export: True          		 # 转出模型
  inference: False		 
  eval_perf: True
 
 
parameters:
  caffe:
    model: './mobilenet_v2.prototxt'
    proto: 'caffe' #lstm_caffe
    blobs: './mobilenet_v2.caffemodel'
  
  tensorflow:
    tf_pb: './ssd_mobilenet_v1_coco_2017_11_17.pb'
    inputs: ['FeatureExtractor/MobilenetV1/MobilenetV1/Conv2d_0/BatchNorm/batchnorm/mul_1']
    outputs: ['concat', 'concat_1']
    input_size_list: [[300, 300, 3]]
 
  tflite:
    model: './sample/tflite/mobilenet_v1/mobilenet_v1.tflite'
 
  onnx:      # 填写要转换模型的model
    model: './best_noop.onnx'   #best_op.onnx   #best_noop.onnx
 
    #C:\Users\HP\Desktop\CODE\yolov5_for_rknn-master\weights\best.onnx
 
  darknet:
    model: './yolov3-tiny.cfg'
    weight: './yolov3.weights'
 
  pytorch:
    model: './yolov5.pt'
    input_size_list: [[3, 512, 512]]
 
  mxnet:
    symbol: 'resnext50_32x4d-symbol.json'
    params: 'resnext50_32x4d-4ecf62e2.params'
    input_size_list: [[3, 224, 224]]
 
  rknn:
    path: './bestrk.rknn'
 
config:
  #mean_value: [[0,0,0]]
  #std_value: [[58.82,58.82,58.82]]
  channel_mean_value: '0 0 0 255' # 123.675 116.28 103.53 58.395 # 0 0 0 255
  reorder_channel: '0 1 2' # '2 1 0'
  need_horizontal_merge: False
  batch_size: 1
  epochs: -1
  target_platform: ['rk3399pro']
  quantized_dtype: 'asymmetric_quantized-u8'
#asymmetric_quantized-u8,dynamic_fixed_point-8,dynamic_fixed_point-16
  optimization_level: 3
 
build:
  do_quantization: True
  dataset: './dataset.txt' # '/home/zen/rknn_convert/quant_data/hand_dataset/pic_path_less.txt'
  pre_compile: False
 
export_rknn:
  export_path: './best_noop1.rknn'
 
init_runtime:
  target: rk3399pro
  device_id: null
  perf_debug: False
  eval_mem: False
  async_mode: False
 
img: &img
  path: './test2.jpg'
 
inference:
  inputs: *img
  data_type: 'uint8'
  data_format: 'nhwc' # 'nchw', 'nhwc'
  inputs_pass_through: None 
 
eval_perf:
  inputs: *img
  data_type: 'uint8'
  data_format: 'nhwc'
  is_print: True
```

其中在设置config部分的参数时，建议看看官方的API介绍，去选择相应的参数部分，在文件`Rockchip_User_Guide_RKNN_Toolkit_V1.6.1_CN.pdf`。

#### RK3399Pro中模型推理

在detect文件夹下，其中data/image下面放的是需要检测的图片，在models文件夹下放的是转换的RKNN模型。

最后点开shell 执行：
`python rknn_detect_for_yolov5_original.py`。
即可在开发板中会生成模型推理的结果和时间。

**推理的时间比较快，60毫秒左右，这个推理速度和我在笔记本电脑（3060）上使用模型detect的速度是差不多的。**

#### 模型预编译


解决模型的加载时间过长的问题

```python
from rknn.api import RKNN
if __name__ == '__main__':
    # Create RKNN object
    rknn = RKNN()
# Load rknn model
ret = rknn.load_rknn('./best_as_200.rknn')
if ret != 0:
    print('Load RKNN model failed.')
    exit(ret)
# init runtime
ret = rknn.init_runtime(target='rk3399pro', rknn2precompile=True)
if ret != 0:
    print('Init runtime failed.')
    exit(ret)
# Note: the rknn2precompile must be set True when call init_runtime
ret = rknn.export_rknn_precompile_model('./best_pre_compile.rknn')
if ret != 0:
    print('export pre-compile model failed.')
    exit(ret)
rknn.release()
```

转换的环境是在RK3399Pro中，方法很笨但是有效。

**将生成的模型继续使用模型的推理代码，在RK3399Pro中进行预测，模型推理速度50毫秒左右，有20FPS，使用Python接口还是比较快的了。**

## **参考链接**

- [CNN 基础之卷积及其矩阵加速](http://shuokay.com/2016/06/08/convolution/)
- [convnet-benchmarks](https://github.com/soumith/convnet-benchmarks)
- [卷积神经网络CNN总结](http://bigdata.51cto.com/art/201705/538792.htm)
- [卷积总结](https://www.mobibrw.com/2017/7557)
- https://github.com/EASY-EAI/yolov5
- https://github.com/soloIife/yolov5_for_rknn
- https://github.com/ultralytics/yolov5
- https://github.com/rockchip-linux/rknn-toolkit
- https://blog.csdn.net/weixin_42237113/category_10147159.html?spm=1001.2014.3001.5482



