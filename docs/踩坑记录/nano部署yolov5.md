# nano上部署自己训练的yolov5s模型

**nano上部署自己训练的yolov5s模型（cuda配置,pytorch,tensorrt加速一站式解决）**

前言：初始化操作可以参照其他博客，链接我之后也会帖，欢迎大家访问我的CSDN： https://blog.csdn.net/weixin_45454706?spm=1011.2124.3001.5343
## 配置cuda
更新源
转载自[这里](https://blog.csdn.net/broliao/article/details/104767103?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522160665407319725211999432%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=160665407319725211999432&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~top_click~default-1-104767103.pc_first_rank_v2_rank_v28&utm_term=nano%E6%B7%B1%E5%BA%A6%E5%AD%A6%E4%B9%A0&spm=1018.2118.3001.4449)

1.备份

```bash
sudo cp /etc/apt/sources.list /etc/apt/sources.list.bak
sudo gedit /etc/apt/sources.list
```
2.删除所有内容，更换成下面的

```bash
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ bionic main multiverse restricted universe
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ bionic-security main multiverse restricted universe
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ bionic-updates main multiverse restricted universe
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ bionic-backports main multiverse restricted universe
deb-src http://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ bionic main multiverse restricted universe
deb-src http://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ bionic-security main multiverse restricted universe
deb-src http://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ bionic-updates main multiverse restricted universe
deb-src http://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ bionic-backports main multiverse restricted universe
```
nano内置好了cuda,但需要配置环境变量才能使用
打开命令行添加环境变量即可
我这里是cuda10.2大家要根据自己的cuda版本去填写路径

```bash
vi ~/.bashrc
export PATH=/usr/local/cuda-10.2/bin${PATH:+:${PATH}}
export LD_LIBRARY_PATH=/usr/local/cuda-10.2/lib64${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}}
export CUDA_ROOT=/usr/local/cuda
source ~/.bashrc
```

查看是否配置成功

```bash
nvcc -V
```
如果显示出自己的cuda版本号即为配置成功

更新pip
```bash
sudo apt-get install python3-pip python3-dev
python3 -m pip install
```
这里我看这个博客要sudo vim /usr/bin/pip3修改pip3，但是我改了以后pip3 install jtop会报错，改回来就又可以了，大家按需修改吧。
之后的一部分转载自[这里](https://blog.csdn.net/u011119817/article/details/99679350)这篇文章十分详细大家可以去看一看
安装jtop库这个可以监控自己的设备cpugpu工作状态

```bash
sudo -H pip3 install jetson-stats
sudo jtop
```

配置需要用到的库
```bash
sudo apt-get install build-essential make cmake cmake-curses-gui
sudo apt-get install git g++ pkg-config curl
sudo apt-get install libatlas-base-dev gfortran libcanberra-gtk-module libcanberra-gtk3-module
sudo apt-get install libhdf5-serial-dev hdf5-tools 
sudo apt-get install nano locate screen
```
安装所需要的依赖环境
```bash
sudo apt-get install libfreetype6-dev 
sudo apt-get install protobuf-compiler libprotobuf-dev openssl
sudo apt-get install libssl-dev libcurl4-openssl-dev
sudo apt-get install cython3
```
安装opencv的系统级依赖,一些编解码的库：
```bash
sudo apt-get install libavcodec-dev libavformat-dev libswscale-dev
sudo apt-get install libxvidcore-dev libavresample-dev
sudo apt-get install libtiff-dev libjpeg-dev libpng-dev
```
更新CMake，这一步是必须的，因为arm架构的很多东西都要从源码编译

```bash
wget http://www.cmake.org/files/v3.13/cmake-3.13.0.tar.gz  #下载，版本号可到官网上查
tar xpvf cmake-3.13.0.tar.gz cmake-3.13.0/  #解压
cd cmake-3.13.0/
./bootstrap --system-curl
make -j4 #编译
echo 'export PATH=~/cmake-3.13.0/bin/:$PATH' >> ~/.bashrc
source ~/.bashrc #更新.bashrc
```
这里可以安装python的虚拟环境进行管理（有一个arm架构下的anaconda但是我们学长说不好用，我也就没去深入研究），这里我还是在主环境下




## 配置pytorch,pytorchvision,cmake,

下载pytorch
cuda10.2对应的torch版本是1.6.0，大家按照自己的版本去下载对应的torch，否则会报错，组长也提供了轮子大家去取即可

```bash
pip3 installtorch-1.6.0a0+b31f58d-cp36-cp36m-linux_aarch64.whl
```
会显示这个错误

![显示这个错误](https://img-blog.csdnimg.cn/20201209202931381.jpg#pic_center)

检查是否成功安装

```bash
python3
import torch
```
如果没有报错即成功安装
下载torchvision
torch1.6.0对应的torchvision版本是0.7.0
由于直接pip的torchvision版本太低所以要从源码编译

```bash
sudo apt-get install libjpeg-dev zlib1g-dev
git clone --branch v0.7.0 https://github.com/pytorch/vision torchvision
cd torchvision
export BUILD_VERSION=0.7.0
sudo python3 setup.py install 
```

同样的方法查看是否成功安装

```bash
python3
import torchvision
```

## 训练模型
yolov5s的训练，这里可以参照其他博文
我建议先在主机上训练完成在拷到nano上，靠nano训练可能不太行

## yolov5s部署
clone yolov5源码
安装requirement里的所需要的环境，注意要用pip3，按需下载，看自己少啥就下啥，如果之后要转onnx也要下载onnx库

```powershell
pip3 install matplotlib==3.2.2
pip3 install scipy==1.4.1.
pip3 install numpy==1.19.4
pip3 install cython
```

可以先在nano上预测一下图片，效果还可以，启动模型要很久，预测效果还可以

```bash
cd yolov5-master
python3 detection.py --source /path/to/xxx.jpg --weights /path/to/best.pt --conf-thres 0.7
```
之后就可以在自己的inference中的output中看到自己预测的图片了。
接着调用摄像头进行视频预测，由于我用的是大恒的usb相机（600，800），用的是大恒官方的api,加入到infer.py中效果还行，大概5fps,效果是一卡一卡的ppt

## tensorrt加速
这才是重头戏！
nano有自带的tensorrt,接下来我们安装pycuda

```powershell
pip3 install pycuda
```

这时我们要用到一个大佬的开源：

```bash
git clone https://github.com/wang-xinyu/tensorrtx.git
```

按照readme中的教程就可以开心地进行模型转换了！我把官方的教程贴到了下面，大家也可以去readme文件中详细了解
注意！：
1.将头文件中的class_num更改为自己的类别数
2.建议把自己的.pt文件重命名为yolov5s.pt，当然如果你要用自己名字别忘了改源码中的对应名字（如果要更改输入图像大小也要在头文件yololayer.h以及python预测yolov5_trt.py中更改)
3.yolov5_trt源码中的图片路径也要改，可以把自己的摄像头调用加进去预测
```markup
How to Run, yolov5s as example
1. generate yolov5s.wts from pytorch with yolov5s.pt, or download .wts from model zoo

git clone https://github.com/wang-xinyu/tensorrtx.git
git clone https://github.com/ultralytics/yolov5.git
// download its weights 'yolov5s.pt'
// copy tensorrtx/yolov5/gen_wts.py into ultralytics/yolov5
// ensure the file name is yolov5s.pt and yolov5s.wts in gen_wts.py
// go to ultralytics/yolov5
python gen_wts.py
// a file 'yolov5s.wts' will be generated.

2. build tensorrtx/yolov5 and run

// put yolov5s.wts into tensorrtx/yolov5
// go to tensorrtx/yolov5
// ensure the macro NET in yolov5.cpp is s
mkdir build
cd build
cmake ..
make
sudo ./yolov5 -s             // serialize model to plan file i.e. 'yolov5s.engine'
sudo ./yolov5 -d  ../samples // deserialize plan file and run inference, the images in samples will be processed.

3. check the images generated, as follows. _zidane.jpg and _bus.jpg

4. optional, load and run the tensorrt model in python

// install python-tensorrt, pycuda, etc.
// ensure the yolov5s.engine and libmyplugins.so have been built
python yolov5_trt.py
```

模型在载入的时候会很慢，并显示waring内存不足，这个一直没有想到好的解决办法，大家如果知道可以告诉我-_-
python版视频预测从之前的5fps提升到了20fps,人眼看不出差别了，基本可以做到实时。
c++部署的视频预测载入模型与预测都比python快，25fps，且没有内存不足的warning！

