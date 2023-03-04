---
layout: post
title: Pytorch入门教程总结学习笔记
date: 2023-03-04
author: lau
tags: [Pytorch, Blog]
comments: true
toc: false
pinned: false


---

Pytorch入门教程总结学习笔记。

<!-- more -->

# Pytorch入门教程

## 1 pytorch构建模型的整个流程

```python
"""
流程：
1.使用torchvision加载并预处理数据集
2.定义网络
3.定义损失函数和优化器
4.训练网络并更新网络参数
5.测试网络
"""
import torch as t
import torchvision as tv
import torchvision.transforms as transforms
from torchvision.transforms import ToPILImage
import matplotlib.pyplot as plt
import torch.nn as nn
import torch.nn.functional as F
import torch.optim as optim
from torch.autograd import Variable

show = ToPILImage() # 可以将tensor转换成Image,方便可视化
#print(show)

# 加载数据集并对数据集预处理
# 定义对数据的预处理(转为tensor,以及归一化)
transform = transforms.Compose([transforms.ToTensor(),transforms.Normalize((0.5,0.5,0.5),(0.5,0.5,0.5))])

# 训练集
trainset = tv.datasets.CIFAR10(root='D:\pytorch',train=True,download=False,transform=transform)
trainloader = t.utils.data.DataLoader(trainset,batch_size=4,shuffle=True,num_workers=2)


#测试集
testset = tv.datasets.CIFAR10(root='D:\pytorch',train=False,download=False,transform=transform)
testloader = t.utils.data.DataLoader(testset,batch_size=4,shuffle=False,num_workers=2)

classes = ('plane','car','bird','cat','deer','dog','frog','horse','ship','truck')

#(data,label) = trainset[100]
#print(classes[label])
#print(data)
#show((data+1)/2).resize((100,100))

#定义网络
class Net(nn.Module):
    def __init__(self):
        # nn.Moulde子类的函数必须在构造函数中执行父类的构造函数
        # 下式等价于nn.Mould.__init__(self)
        super(Net,self).__init__()
        self.conv1 = nn.Conv2d(3,6,5) # 卷积层3表示输入图片为3通道，6表示输出通道数，5表示卷积核为5*5
        # 卷积层
        self.conv2 = nn.Conv2d(6,16,5)
        # 全连接层 y=wx+b
        self.fc1 = nn.Linear(16*5*5,120)
        self.fc2 = nn.Linear(120,84)
        self.fc3 = nn.Linear(84,10)

    def forward(self,x):
        # 卷积 -> 激活 -> 池化
        x = F.max_pool2d(F.relu(self.conv1(x)),(2,2))
        x = F.max_pool2d(F.relu(self.conv2(x)),2)
        # reshape,-1表示自适应
        x = x.view(x.size()[0],-1)
        x = F.relu(self.fc1(x))
        x = F.relu(self.fc2(x))
        x = self.fc3(x)
        return x
if __name__ == '__main__':
    net = Net()
    #print(net)
    # 定义损失函数和优化器
    criterion = nn.CrossEntropyLoss()
    optimizer = optim.SGD(net.parameters(),lr=0.001,momentum=0.9)


    # 训练网络（包括输入数据，前向传播+反向传播，更新参数）
    for epoch in range(2):
        running_loss=0.0
        for i,data in enumerate(trainloader,0):
            #输入数据
            #print(data)
            inputs,lables = data
            inputs,lables = Variable(inputs),Variable(lables)
            #梯度清零
            optimizer.zero_grad()
            #前向传播+反向传播
            outputs = net(inputs)
            loss = criterion(outputs,lables)
            loss.backward()
            #更新参数
            optimizer.step()
            #打印log信息
            running_loss += loss.item()
            if i%2000 == 1999: # 每2000个batch打印一次训练状态
                print('[%d,%5d] loss:%.3f' % (epoch+1,i+1,running_loss/2000))
                running_loss = 0.0
    print("Finished Training")

    # 开始测试训练效果
    dataiter = iter(testloader)
    images,lables = dataiter.next() # 一个batch返回4张图片
    print('实际的label: ',' '.join('%08s' % classes [lables[j]] for j in range(4)))

    # 接着计算网络预测的lable
    # 计算图片在每个类别上的分数
    outputs = net(Variable(images))
    # 得分最高的那个类
    _,predicted = t.max(outputs.data,1)
    print('预测结果： ',' '.join('%5s'% classes [predicted[j]] for j in range(4)))

    #查看整个测试集上面的效果
    correct = 0 #预测正确的图片数
    total = 0 #总共的图片数
    for data in testloader:
        images,lables = data
        outputs = net(Variable(images))
        _, predicted = t.max(outputs.data, 1)
        total += lables.size(0)
        correct += (predicted == lables).sum()
    print('10000张测试集中的准确率：%d %%' % (100*correct/total))

```

## 2 pytorch入门：调用类的内置函数和自定义函数的区别

```python
from torch.utils.tensorboard import SummaryWriter
"""
1.简介：
tensorboard是tensorflow内置的一个可视化工具，它通过将tensorflow程序输出的日志文件的信息可视化使得tensorflow程序的理解、
调试和优化更加简单高效。Tensorboard的可视化依赖于tensorflow程序运行输出的日志文件，因而tensorboard和tensorflow程序在不同的进程中运行。
2.pytorch如何使用tensorboard:
2.1 导入模块from torch.utils.tensorboard import SummaryWriter
2.2 创建子文件夹logs:SummaryWriter('logs')
2.3 使用add_scalar开始对自己的程序进行可视化
2.4 运行完程序后，会生成一个logs文件夹，在Terminal并在logs文件夹的绝对路径下运行：tensorboard --logdir=logs,进入输出的url下即可看到程序的可视化图
3.如果想将可视化图输出到不同的图上，需要通过SummaryWriter('新建文件夹')或者将logs下面的一些文件进行清理，重新生成新的可视化图
"""

writer = SummaryWriter('logs')

for i in range(100):
    writer.add_scalar('y=2x',3*i,i)  #title/y/x

writer.close()
```

## 3 pytorch入门：学会使用tensorboard如何可视化照片

```python
"""
本节主要学会使用tensorboard如何可视化照片
"""

from torch.utils.tensorboard import SummaryWriter
import numpy as np
from PIL import Image

writer = SummaryWriter('add_image_logs')
image_path = 'dataset/train/ants/0013035.jpg'
img_PIL = Image.open(image_path)
img_array = np.array(img_PIL)

writer.add_image('test',img_array,1,dataformats='HWC')  #从PIL到numpy的转换需要指定三个值的HWC的位置
for i in range(100):
    writer.add_scalar('y=2x',3*i,i)  #title/y/x

writer.close()
```

## 4 pytorch入门：学会如何读取已经下载好的数据集

```python
"""
本节主要学会如何读取已经下载好的数据集
"""

from torch.utils.data import Dataset
from PIL import Image
import os

class MyData(Dataset):
    def __init__(self,root_dir,label_dir):
        self.root_dir = root_dir
        self.label_dir = label_dir
        self.path = os.path.join(self.root_dir,self.label_dir)
        self.img_path = os.listdir(self.path)

    def __getitem__(self, idx):
        img_name = self.img_path[idx]
        img_item_path = os.path.join(self.path,img_name)
        img = Image.open(img_item_path)
        label = self.label_dir
        return img,label

    def __len__(self):
        return len(self.img_path)

root_dir = 'hymenoptera_data/train'
ant_label_dir = 'ants'
bees_label_dir = 'bees'
ant_dataset = MyData(root_dir,ant_label_dir)
bees_dataset = MyData(root_dir,bees_label_dir)

train_dataset = ant_dataset + bees_dataset
```

## 5 pytorch入门：学会如何对已经下载好的数据集进行重命名，并分割出特征

```python
"""
本节主要学会如何对已经下载好的数据集进行重命名，并分割出特征
"""

import os

root_dir = 'hymenoptera_data/train'
target_dir = 'bees_image'
image_path = os.listdir(os.path.join(root_dir,target_dir))
label = target_dir.split('_')[0]
out_dir = 'bees_label'
for i in image_path:
    file_name = i.split('.jpg')[0]
    with open(os.path.join(root_dir,out_dir,'{}.txt'.format(file_name)),'w') as f:
        f.write(label)
```

## 6 pytorch入门：学会如何下载数据集，并通过tensorboard查看下载的数据集

```python
"""
本节学会如何下载数据集，并通过tensorboard查看下载的数据集
"""

import torchvision
import ssl
from torch.utils.tensorboard import SummaryWriter

ssl._create_default_https_context = ssl._create_unverified_context  # 全局取消证书验证

# Compose中存放transforms列表，列表中有totensor和resize两种方法
dataset_transfroms = torchvision.transforms.Compose([torchvision.transforms.ToTensor(),torchvision.transforms.Resize((512,512))])

#这个时候也可以按着Ctrl鼠标点击CIFAR10进入，往上滑，找到数据集的下载链接去浏览器中下载即可
train_set = torchvision.datasets.CIFAR10(root='./torchvision_dataset',train=True,download=True,transform=dataset_transfroms)
test_set = torchvision.datasets.CIFAR10(root='./torchvision_dataset',train=False,download=True,transform=dataset_transfroms)

#print(train_set[0])

# img,target = train_set[0]
# print(img)
# print(target)
writer = SummaryWriter('dataset_log')
for i in range(10):
    img,target = train_set[i]
    writer.add_image('train_set',img,i)

writer.close()
```

## 7 pytorch入门：学会dataloader的使用

```python
"""
本节主要学会dataloader的使用
"""

import torchvision
from torch.utils.data import DataLoader
from torch.utils.tensorboard import SummaryWriter

test_data = torchvision.datasets.CIFAR10("./torchvision_dataset",train=False,transform=torchvision.transforms.ToTensor())

# drop_lastd代表除以batch_size除不尽的时候是否丢弃余数
test_loder = DataLoader(dataset=test_data,batch_size=4,shuffle=False,num_workers=0,drop_last=False) # 取打乱后的4个一打包放在一起，形成多个打包

img,target = test_data[0]
print(img.shape)
print(target) # target代表的是标签值
writer = SummaryWriter('test_dataloader')
for epoch in range(2):
    step = 0
    for data in test_loder:
        imgs,targets = data
        # print(imgs.shape)
        # print(targets)
        writer.add_images("Epoch {}".format(epoch),imgs,step)
        step+=1
writer.close()
```

## 8 pytorch入门：学会transform的使用

```python
"""
本节主要学会transform的使用
"""

from PIL import Image
from torchvision import transforms
from torch.utils.tensorboard import SummaryWriter
"""
transforms是对数据进行一系列转换的模块
比如输入一张图片->transforms工具箱（totensor/resize等等）->结果
"""
"""
transforms在python中的使用->tensor数据类型
通过transforms.ToTensor来看两个问题
1.transforms该如何使用
2.为什么要使用Tensor数据类型
"""

img_path = 'dataset/train/ants/0013035.jpg'
img = Image.open(img_path)
writer = SummaryWriter('transforms_log')

# 熟悉ToTensor方法的运用
trans_tensor = transforms.ToTensor()  # 首先定义一个ToTensor的对象
tensor_img = trans_tensor(img)
#print(tensor_img)
writer.add_image('tensor',tensor_img)
# 使用opencv
# import cv2
# cv_img = cv2.imread(img_path)

# 熟悉Normalize(归一化)的运用：(样本值-均值)/标准差
print(tensor_img[0][0][0])
trans_normalize = transforms.Normalize([0.5,0.5,0.5],[0.5,0.5,0.5])  #因为图片是三维的，所以需要输入三个均值和标准差
normalize_img = trans_normalize(tensor_img)
print(normalize_img[0][0][0])
writer.add_image('Normalize',normalize_img)

# 熟悉Resize的运用
print(img.size)
trans_resize = transforms.Resize((512,512))
# img PIL --> resize --> resize_img PIL
resize_img = trans_resize(img)
# resize_img PIL --> resize --> resize_img tensor
resize_img = trans_tensor(resize_img)
writer.add_image("resize",resize_img)

# 熟悉Compose的运用
trans_resize_2 = transforms.Resize(512)
compose_trans = transforms.Compose([trans_resize_2,trans_tensor])
img_resize_2 = compose_trans(img)
writer.add_image("compose",img_resize_2)

# 熟悉randomcrop
trans_random = transforms.RandomCrop((500,512))
trans_compose_2 = transforms.Compose([trans_random,trans_tensor])
for i in range(10):
    img_crop = trans_compose_2(img)
    writer.add_image('randomcup',img_crop,i)

writer.close()
```

## 9 pytorch入门：学会nn.Module的使用

```python
"""
本节主要学会nn.Module的使用
"""

import torch
from torch import nn
from torch.autograd.grad_mode import F
class Zkl(nn.Module):
    def __init__(self):
        super().__init__()

    def forward(self,x):
        x = x+1
        return x

zkl = Zkl()
c=torch.Tensor(1)
b=zkl(c)
print(b)
```

## 10 pytorch入门：F.Conv2d和nn.Conv2d

```python
import torch
import torch.nn.functional as F
# 小括号里面有几个[]就代表是几维数据
input = torch.tensor([[1,2,0,3,1],
                      [0,1,2,3,1],
                      [1,2,1,0,0],
                      [5,2,3,1,1],
                      [2,1,0,1,1]])

kernel = torch.tensor([[1,2,1],
                       [0,1,0],
                       [2,1,0]])

input = torch.reshape(input,(1,1,5,5))
kernel = torch.reshape(kernel,(1,1,3,3))

# stride代表的是步长的意思，即每次卷积核向左或者向下移动多少步进行相乘
#  因为conv2d的input和weight对应的tensor是[batch,channel,h,w],所以上述才将它们进行reshape
output = F.conv2d(input,kernel,stride=1)
print(output)

output = F.conv2d(input,kernel,stride=2)
print(output)

# padding代表的是向上下左右填充的行列数，里面数字填写0
output3 = F.conv2d(input,kernel,stride=1,padding=1)
print(output3)
```

## 11 pytorch入门：最大池化层的使用

```python
"""
本节主要学会最大池化层的使用
池化层大大减少了数据量
"""

import torch
import torchvision
from torch import nn
from torch.nn import Conv2d, MaxPool2d
from torch.utils.data import DataLoader
from torch.utils.tensorboard import SummaryWriter

dataset = torchvision.datasets.CIFAR10('./torchvision_dataset', train=False, download=False,
                                       transform=torchvision.transforms.ToTensor())
dataloader = DataLoader(dataset,batch_size=64)
# input = torch.tensor([[1,2,0,3,1],
#                       [0,1,2,3,1],
#                       [1,2,1,0,0],
#                       [5,2,3,1,1],
#                       [2,1,0,1,1]],dtype=torch.float32)
# input = torch.reshape(input,(-1,1,5,5))

class Zkl(nn.Module):
    def __init__(self):
        super(Zkl, self).__init__()
        self.maxpool1 =MaxPool2d(kernel_size=3,ceil_mode=False)

    def forward(self,x):
        output = self.maxpool1(x)
        return output

zkl = Zkl()
writer = SummaryWriter('nn_maxpool')
step = 0
for data in dataloader:
    imgs,targets = data
    writer.add_images('input_maxpool',imgs,step)
    output = zkl(imgs)
    writer.add_images('output_maxpool',output,step)
    step += 1

writer.close()
```

## 12 pytorch入门：非线性激活层的使用

```python
import torch
import torchvision
from torch import nn

# input = torch.tensor([[1,-0.5],
#                       [-1,3]],dtype=torch.float32)
#
# #print(input.shape)
#
# input = torch.reshape(input,(-1,1,2,2))
# #print(input.shape)
from torch.utils.data import DataLoader
from torch.utils.tensorboard import SummaryWriter

dataset = torchvision.datasets.CIFAR10('',train=False,download=False,transform=torchvision.transforms.ToTensor())
dataloader = DataLoader(dataset,batch_size=64)





class Zkl(nn.Module):
    def __init__(self):
        super(Zkl, self).__init__()
        self.nn_relu = nn.ReLU(inplace = True)  # 这里的inplace代表的意思就是生成的值是否对原变量的进行替换
        self.nn_sigmod = nn.Sigmoid()

    def forward(self,input):
        output = self.nn_sigmod(input)
        return output

zkl = Zkl()
writer = SummaryWriter('nn_sigmoid_log')
step = 0
for data in dataloader:
    imgs,targets = data
    writer.add_images('relu_input',imgs,step)
    output = zkl(imgs)
    writer.add_images('relu_output',output,step)
    step += 1

writer.close()
```

## 13 pytorch入门：线性层的使用

```python
import torch
import torchvision
from torch import nn
from torch.utils.data import DataLoader
from torch.utils.tensorboard import SummaryWriter

dataset = torchvision.datasets.CIFAR10('',train=False,download=False,transform=torchvision.transforms.ToTensor())
dataloader = DataLoader(dataset,batch_size=64,drop_last=True)

class Zkl(nn.Module):
    def __init__(self):
        super(Zkl, self).__init__()
        self.nn_linear = nn.Linear(196608,10)

    def forward(self,input):
        output = self.nn_linear(input)
        return output

zkl = Zkl()
writer = SummaryWriter('nn_linear_log')
step = 0
for data in dataloader:
    imgs,targets = data
    # writer.add_images('linear_input',imgs,step)
    # output = zkl(imgs)
    # writer.add_images('linear_output',output,step)
    # step += 1
    print(imgs.shape)
    output = torch.flatten(imgs)
    print(output.shape)
    output = zkl(output)
    print(output.shape)
writer.close()
```

## 14 pytorch入门：Sequential的使用

```python
import torch
from torch import nn
from torch.nn import Conv2d, MaxPool2d, Linear, Flatten
from torch.utils.tensorboard import SummaryWriter


class Zkl(nn.Module):
    def __init__(self):
        super(Zkl, self).__init__()
        # self.conv1 = Conv2d(3,32,5,padding=2)
        # self.maxpool1 = MaxPool2d(2)
        # self.conv2 = Conv2d(32,32,5,padding=2)
        # self.maxpool2 = MaxPool2d(2)
        # self.conv3 = Conv2d(32,64,5,padding=2)
        # self.maxpool3 = MaxPool2d(2)
        # self.flatten = Flatten()
        # self.linear1 = Linear(1024,64)
        # self.linear2 = Linear(64,10)

        self.modle1 = nn.Sequential(
            Conv2d(3, 32, 5, padding=2),
            MaxPool2d(2),
            Conv2d(32, 32, 5, padding=2),
            MaxPool2d(2),
            Conv2d(32, 64, 5, padding=2),
            MaxPool2d(2),
            Flatten(),
            Linear(1024, 64),
            Linear(64,10)
        )

    def forward(self,x):
        # x = self.conv1(x)
        # x = self.maxpool1(x)
        # x = self.conv2(x)
        # x = self.maxpool2(x)
        # x = self.conv3(x)
        # x = self.maxpool3(x)
        # x = self.flatten(x)
        # x = self.linear1(x)
        # x = self.linear2(x)
        x = self.modle1(x)
        return x

zkl = Zkl()
#print(zkl)
# 以下是为了测试一下构建的网络是否正确
input = torch.ones((64,3,32,32))
output = zkl(input)
print(output.shape)

writer = SummaryWriter('sequential_log')
writer.add_graph(zkl,input)
writer.close()
```

## 15 pytorch入门：损失函数的使用

```python
import torch
from torch.nn import L1Loss
from torch import nn

input = torch.tensor([1,2,3],dtype=torch.float32)

target = torch.tensor([1,2,5],dtype=torch.float32)

# 使用reshape进行维度的变换
input = torch.reshape(input,(1,1,1,3))  # 代表的是1个样本，通道为1，宽*高=1*3
target = torch.reshape(target,(1,1,1,3))

loss = L1Loss(reduction = 'sum')  # 这里的reduction可以设置为'sum'或者'mean'，并且默认是'mean'
result = loss(input,target)

loss_mea = nn.MSELoss()
result_mea = loss_mea(input,target)
print(result)
print(result_mea)

x = torch.tensor([0.1,0.2,0.3])
y = torch.tensor([1])

x = torch.reshape(x,(1,3))  # 交叉熵的输入格式是（batch_size,类别数量），输出格式（batch_size）
loss = nn.CrossEntropyLoss()
result_cro = loss(x,y)
print(result_cro)
```

## 16 pytorch入门：优化器的使用

```python
import torch.optim
import torchvision
from torch import nn
from torch.nn import Conv2d, MaxPool2d, Flatten, Linear
from torch.utils.data import DataLoader

dataset = torchvision.datasets.CIFAR10('./torchvision_dataset',train=False,download=False,transform=torchvision.transforms.ToTensor())
dataloader = DataLoader(dataset,batch_size=64,drop_last=True)


class Zkl(nn.Module):
    def __init__(self):
        super(Zkl, self).__init__()
        # self.conv1 = Conv2d(3,32,5,padding=2)
        # self.maxpool1 = MaxPool2d(2)
        # self.conv2 = Conv2d(32,32,5,padding=2)
        # self.maxpool2 = MaxPool2d(2)
        # self.conv3 = Conv2d(32,64,5,padding=2)
        # self.maxpool3 = MaxPool2d(2)
        # self.flatten = Flatten()
        # self.linear1 = Linear(1024,64)
        # self.linear2 = Linear(64,10)

        self.modle1 = nn.Sequential(
            Conv2d(3, 32, 5, padding=2),
            MaxPool2d(2),
            Conv2d(32, 32, 5, padding=2),
            MaxPool2d(2),
            Conv2d(32, 64, 5, padding=2),
            MaxPool2d(2),
            Flatten(),
            Linear(1024, 64),
            Linear(64,10)
        )

    def forward(self,x):
        # x = self.conv1(x)
        # x = self.maxpool1(x)
        # x = self.conv2(x)
        # x = self.maxpool2(x)
        # x = self.conv3(x)
        # x = self.maxpool3(x)
        # x = self.flatten(x)
        # x = self.linear1(x)
        # x = self.linear2(x)
        x = self.modle1(x)
        return x
#引用类
zkl = Zkl()
#定义损失函数
loss = nn.CrossEntropyLoss()
#定义优化器
optim = torch.optim.SGD(zkl.parameters(),lr=0.1)
for epoch in range(20):
    running_loss = 0.0
    for data in dataloader:
        #获取输入特征和输入标签
        imgs,targets = data
        #将输入放入模型，获取输出
        output = zkl(imgs)
        #再通过损失函数获取损失值
        loss_cro = loss(output,targets)  # 两个参数为实际输出和目标输出
        #loss_cro.backward()  # 梯度下降法
        #为防止梯度累积，每一次循环都要将每个节点对应的梯度归为0，上一次循环计算出来的梯度对这一次循环没有用
        optim.zero_grad()
        #再获取每个参数可以调节的梯度
        loss_cro.backward()
        #再对每个参数进行调优
        optim.step()
        #最后打印每一次训练的损失值，观察是否在缩小
        #print(loss_cro)
        running_loss += loss_cro
    print(running_loss)
```

## 17 pytorch入门：现有模型的使用和修改

```python
import ssl
"""
这一节主要是学会怎么在现有模型上进行修改模型结构
"""
import torchvision

# ssl._create_default_https_context = ssl._create_unverified_context  # 全局取消证书验证
# dataset = torchvision.datasets.ImageNet('../imagenet_data',split='train',download=True,transform=torchvision.transforms.ToTensor())
from torch import nn
from torch.utils.data import DataLoader

vgg16_fal = torchvision.models.vgg16(pretrained=False) #还需要训练的模型
vgg16_tr = torchvision.models.vgg16(pretrained=True)  #已经训练好的模型
#print('ok')
#print(vgg16_tr)  # vgg16是一个对1000种分类模型

# 现在想将vgg16用到CIFAR10上面，即改为10分类模型
dataset = torchvision.datasets.CIFAR10('./torchvision_dataset',train=False,download=False,transform=torchvision.transforms.ToTensor())
dataloader = DataLoader(dataset,batch_size=64,drop_last=True)

# 方法1：修改vgg16模型，在最后加一个线性层，即输入1000，输出10
vgg16_tr.classifier.add_module('add_linear',nn.Linear(1000,10))
print(vgg16_tr)

# 方法2：直接修改vgg16最后一个线性层输出为10
vgg16_fal.classifier[6] = nn.Linear(4096,10)
print(vgg16_fal)
```

## 18 pytorch入门：构建后的模型保存和加载

```python
"""
这一节学会如何保存模型以及加载模型
"""

import torch
import torchvision
#保存方法1 模型结构+模型参数
#先保存
from torch import nn

vgg16 = torchvision.models.vgg16(pretrained=True)
torch.save(vgg16,'vgg16.pth')
#后加载
# vgg16_load= torch.load('vgg16.pth')
# print(vgg16_load)

#保存方法2 模型参数（官方推荐）
#先保存
torch.save(vgg16.state_dict(),'vggi6_method2.pth')  # 保存成字典形式
#后加载
vgg16.load_state_dict(torch.load('vggi6_method2.pth'))


#实例
class Zkl(nn.Module):
    def __init__(self):
        super(Zkl, self).__init__()
        self.conv1 = nn.Conv2d(3,64,kernel_size=(3,3))

    def forward(self,x):
        x = self.conv1(x)
        return x

#保存
zkl = Zkl()
torch.save(zkl,'zkl_method.pth')

model = torch.load('zkl_method.pth')
print(model)
```

## 19 pytorch入门：模型的完整训练套路

```python
import torch
import torchvision
from torch import nn
from torch.nn import Conv2d, MaxPool2d, Flatten, Linear

from torch.utils.data import DataLoader
# 1.准备数据集并加载
from torch.utils.tensorboard import SummaryWriter

train = torchvision.datasets.CIFAR10('./torchvision_dataset',train=True,download=False,transform=torchvision.transforms.ToTensor())
test = torchvision.datasets.CIFAR10('./torchvision_dataset',train=False,download=False,transform=torchvision.transforms.ToTensor())
train_loader = DataLoader(train,batch_size=64,drop_last=True)
test_loader = DataLoader(test,batch_size=64,drop_last=True)

#获取两个数据集的长度
train_len = len(train)
test_len = len(test)

#定义网络结构
class Zkl(nn.Module):
    def __init__(self):
        super(Zkl, self).__init__()
        # self.conv1 = Conv2d(3,32,5,padding=2)
        # self.maxpool1 = MaxPool2d(2)
        # self.conv2 = Conv2d(32,32,5,padding=2)
        # self.maxpool2 = MaxPool2d(2)
        # self.conv3 = Conv2d(32,64,5,padding=2)
        # self.maxpool3 = MaxPool2d(2)
        # self.flatten = Flatten()
        # self.linear1 = Linear(1024,64)
        # self.linear2 = Linear(64,10)

        self.modle1 = nn.Sequential(
            Conv2d(3, 32, 5, padding=2),
            MaxPool2d(2),
            Conv2d(32, 32, 5, padding=2),
            MaxPool2d(2),
            Conv2d(32, 64, 5, padding=2),
            MaxPool2d(2),
            Flatten(),
            Linear(1024, 64),
            Linear(64,10)
        )

    def forward(self,x):
        # x = self.conv1(x)
        # x = self.maxpool1(x)
        # x = self.conv2(x)
        # x = self.maxpool2(x)
        # x = self.conv3(x)
        # x = self.maxpool3(x)
        # x = self.flatten(x)
        # x = self.linear1(x)
        # x = self.linear2(x)
        x = self.modle1(x)
        return x

#定义模型对象
zkl = Zkl()

#定义损失函数
loss = nn.CrossEntropyLoss()

#定义优化器
learn_rate = 1e-2
optim = torch.optim.SGD(zkl.parameters(),lr=learn_rate)

#设置训练网络的一些参数
#记录训练的次数
total_train_step = 0
#记录测试的次数
total_test_step = 0

#定义训练轮数
epoch=10

#添加tensorboard
writer = SummaryWriter('train_log')

for i in range(epoch):
    print(f"-------第 {i} 轮训练开始----------")
    zkl.train() # 设置这个是使模型进入一个特定状态，只对dropout层或者batchnorml层有作用，并且有这些层就必须调用
    for data in train_loader:
        imgs,targets = data
        output = zkl(imgs)
        train_loss=loss(output,targets)
        #优化器调优模型
        optim.zero_grad()
        train_loss.backward()
        optim.step()

        total_train_step += 1
        if total_train_step%100 == 0:  # 减少打印次数，防止看不清楚结果
            print(f"训练次数 {total_train_step}：loss={train_loss.item()}")  # 加上.item()是为了使输出的是一个值，而不是tensor(1)这样的表达形式
            writer.add_scalar('model_train',train_loss,total_train_step)
    #每一轮训练完后开始测试
    zkl.eval()  # 设置这个是使模型进入一个特定状态，只对dropout层或者batchnorml层有作用，并且有这些层就必须调用
    total_test_loss = 0
    total_accuracy = 0
    with torch.no_grad():  # 不进行优化器在测试过程中，只需要使用训练好的参数
        for data in test_loader:
            imgs,targets = data
            output = zkl(imgs)
            loss_test = loss(output,targets)
            total_test_loss += loss_test.item()
            accuracy = (output.argmax(1) == targets).sum()
            total_accuracy += accuracy

    print(f"第 {i} 轮训练后测试的loss: {total_test_loss}")
    print(f"第 {i} 轮训练后测试的accuracy: {total_accuracy/test_len}")
    writer.add_scalar('model_test', loss_test, total_test_step)
    writer.add_scalar('model_accuracy',total_accuracy/test_len,total_test_step)
    total_test_step += 1

    #保存每一轮训练的结果
    torch.save(zkl,f'modle_train_{i}.pth')
    print('模型已保存')

writer.close()
```

## 20 pytorch入门：准确率的使用

```python
import torch

output_1 = torch.tensor([[0.1,0.2],
                      [0.05,0.4]],dtype=torch.float32)

print(output_1.argmax(0)) # 参数设置为0，通过竖向寻找最大值，并获得最大值对应的索引

output_2 = torch.tensor([[0.1,0.2],
                      [0.3,0.4]],dtype=torch.float32)

print(output_2.argmax(1)) # 参数设置为1，通过横向寻找最大值，并获得最大值对应的索引

predict = output_2.argmax(1)
target = torch.tensor([0,1])
print((predict == target).sum())  # 统计预测准确的数量
```

## 21 pytorch入门：在gpu上训练模型

```python
"""
网络模型，数据(输入，target)，损失函数可以使用gpu跑，直接调用.cuda()即可

可以尝试使用Google Colab——用谷歌免费GPU跑你的深度学习代码
"""

import torch
import torchvision
from torch import nn
from torch.nn import Conv2d, MaxPool2d, Flatten, Linear

from torch.utils.data import DataLoader
# 1.准备数据集并加载
from torch.utils.tensorboard import SummaryWriter

train = torchvision.datasets.CIFAR10('./torchvision_dataset',train=True,download=False,transform=torchvision.transforms.ToTensor())
test = torchvision.datasets.CIFAR10('./torchvision_dataset',train=False,download=False,transform=torchvision.transforms.ToTensor())
train_loader = DataLoader(train,batch_size=64,drop_last=True)
test_loader = DataLoader(test,batch_size=64,drop_last=True)

#获取两个数据集的长度
train_len = len(train)
test_len = len(test)

# 定义训练的设备
#device = torch.device('cuda:0')
device = torch.device('cpu')

#定义网络结构
class Zkl(nn.Module):
    def __init__(self):
        super(Zkl, self).__init__()
        # self.conv1 = Conv2d(3,32,5,padding=2)
        # self.maxpool1 = MaxPool2d(2)
        # self.conv2 = Conv2d(32,32,5,padding=2)
        # self.maxpool2 = MaxPool2d(2)
        # self.conv3 = Conv2d(32,64,5,padding=2)
        # self.maxpool3 = MaxPool2d(2)
        # self.flatten = Flatten()
        # self.linear1 = Linear(1024,64)
        # self.linear2 = Linear(64,10)

        self.modle1 = nn.Sequential(
            Conv2d(3, 32, 5, padding=2),
            MaxPool2d(2),
            Conv2d(32, 32, 5, padding=2),
            MaxPool2d(2),
            Conv2d(32, 64, 5, padding=2),
            MaxPool2d(2),
            Flatten(),
            Linear(1024, 64),
            Linear(64,10)
        )

    def forward(self,x):
        # x = self.conv1(x)
        # x = self.maxpool1(x)
        # x = self.conv2(x)
        # x = self.maxpool2(x)
        # x = self.conv3(x)
        # x = self.maxpool3(x)
        # x = self.flatten(x)
        # x = self.linear1(x)
        # x = self.linear2(x)
        x = self.modle1(x)
        return x

#定义模型对象
zkl = Zkl()
#放在gpu上跑，调用cuda
# if torch.cuda.is_available():
#     zkl = zkl.cuda()
zkl = zkl.to(device)  # 将我们的网络转移到设备上面

#定义损失函数
loss = nn.CrossEntropyLoss()
#放在gpu上跑，调用cuda
# if torch.cuda.is_available():
#     loss = loss.cuda()
loss = loss.to(device)

#定义优化器
learn_rate = 1e-2
optim = torch.optim.SGD(zkl.parameters(),lr=learn_rate)

#设置训练网络的一些参数
#记录训练的次数
total_train_step = 0
#记录测试的次数
total_test_step = 0

#定义训练轮数
epoch=10

#添加tensorboard
writer = SummaryWriter('train_log')

for i in range(epoch):
    print(f"-------第 {i} 轮训练开始----------")
    zkl.train() # 设置这个是使模型进入一个特定状态，只对dropout层或者batchnorml层有作用，并且有这些层就必须调用
    for data in train_loader:
        imgs,targets = data
        # if torch.cuda.is_available():
        #     imgs = imgs.cuda()
        #     targets = targets.cuda()
        imgs = imgs.to(device)
        targets = targets.to(device)
        output = zkl(imgs)
        train_loss=loss(output,targets)
        #优化器调优模型
        optim.zero_grad()
        train_loss.backward()
        optim.step()

        total_train_step += 1
        if total_train_step%100 == 0:  # 减少打印次数，防止看不清楚结果
            print(f"训练次数 {total_train_step}：loss={train_loss.item()}")  # 加上.item()是为了使输出的是一个值，而不是tensor(1)这样的表达形式
            writer.add_scalar('model_train',train_loss,total_train_step)
    #每一轮训练完后开始测试
    zkl.eval()  # 设置这个是使模型进入一个特定状态，只对dropout层或者batchnorml层有作用，并且有这些层就必须调用
    total_test_loss = 0
    total_accuracy = 0
    with torch.no_grad():  # 不进行优化器在测试过程中，只需要使用训练好的参数
        for data in test_loader:
            imgs,targets = data
            # if torch.cuda.is_available():
            #     imgs = imgs.cuda()
            #     targets = targets.cuda()
            imgs = imgs.to(device)
            targets = targets.to(device)
            output = zkl(imgs)
            loss_test = loss(output,targets)
            total_test_loss += loss_test.item()
            accuracy = (output.argmax(1) == targets).sum()
            total_accuracy += accuracy

    print(f"第 {i} 轮训练后测试的loss: {total_test_loss}")
    print(f"第 {i} 轮训练后测试的accuracy: {total_accuracy/test_len}")
    writer.add_scalar('model_test', loss_test, total_test_step)
    writer.add_scalar('model_accuracy',total_accuracy/test_len,total_test_step)
    total_test_step += 1

    #保存每一轮训练的结果
    torch.save(zkl,f'modle_train_{i}.pth')
    print('模型已保存')

writer.close()
```

## 22 pytorch入门：完整的模型验证套路

```python
"""
目的：输入一张验证图片，定义之前训练时的网络结构，加载训练好的模型，验证模型是否准确预测输入图片的类别即可

"""

import torch
import torchvision
from PIL import Image
from torch import nn
from torch.nn import Conv2d, MaxPool2d, Linear, Flatten

image_path = r'./dataset/train/ants/0013035.jpg'

image = Image.open(image_path)
print(image)

image = image.convert('RGB')  # 因为png格式是四个通道（还拥有一个透明通道），调用这个语句是只保留三个通道，如果图片只有三个通道，调用这个语句也不会改变什么，但是这条语句能让我们适应各种图片格式

#开始改变照片尺寸，resize
transform = torchvision.transforms.Compose([torchvision.transforms.Resize((32,32)),
                                            torchvision.transforms.ToTensor()])  # compose是将一系列的变换放在一起执行

image = transform(image)
print(image.shape)

#定义网络结构
class Zkl(nn.Module):
    def __init__(self):
        super(Zkl, self).__init__()
        # self.conv1 = Conv2d(3,32,5,padding=2)
        # self.maxpool1 = MaxPool2d(2)
        # self.conv2 = Conv2d(32,32,5,padding=2)
        # self.maxpool2 = MaxPool2d(2)
        # self.conv3 = Conv2d(32,64,5,padding=2)
        # self.maxpool3 = MaxPool2d(2)
        # self.flatten = Flatten()
        # self.linear1 = Linear(1024,64)
        # self.linear2 = Linear(64,10)

        self.modle1 = nn.Sequential(
            Conv2d(3, 32, 5, padding=2),
            MaxPool2d(2),
            Conv2d(32, 32, 5, padding=2),
            MaxPool2d(2),
            Conv2d(32, 64, 5, padding=2),
            MaxPool2d(2),
            Flatten(),
            Linear(1024, 64),
            Linear(64,10)
        )

    def forward(self,x):
        # x = self.conv1(x)
        # x = self.maxpool1(x)
        # x = self.conv2(x)
        # x = self.maxpool2(x)
        # x = self.conv3(x)
        # x = self.maxpool3(x)
        # x = self.flatten(x)
        # x = self.linear1(x)
        # x = self.linear2(x)
        x = self.modle1(x)
        return x

#加载保存的模型
model = torch.load('modle_train_9.pth')
print(model)
image = torch.reshape(image,(1,3,32,32))
#使加载的模型规定为测试模型
model.eval()
#
with torch.no_grad(): # 没有梯度优化，节约性能
    output = model(image)
print(output)
print(output.argmax(1))
```

