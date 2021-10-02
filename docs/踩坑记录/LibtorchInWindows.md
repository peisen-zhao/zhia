# Windows配置libtorch

## Visual Studio配置libtorch

### 官网下载解压包 

网址：https://pytorch.org/

![image-20210201164835477](C:\Users\Peisen Zhao\AppData\Roaming\Typora\typora-user-images\image-20210201164835477.png)

### 需要和原本pytorch,cuda版本对应，解压

### 打开vs新建项目，新建属性表

#### （1）C/C++->常规->附加包含目录里面添加以下内容

![image-20210201160947578](C:\Users\Peisen Zhao\AppData\Roaming\Typora\typora-user-images\image-20210201160947578.png)

```c++
xxx\libtorch-win-shared-with-deps-latest\include\torch\csrc\api\include
xxx\libtorch-win-shared-with-deps-latest\include
```



#### （2）链接器->常规->附加库目录

```
path_to\libtorch\lib
```



#### 			（3）输入->附加依赖项里面添加libtorch/lib 中所有的lib文件

### debugx64属性

#### （1）调试->环境->

```
PATH=path_to\libtorch\lib;%PATH%
```

#### （2）c/c++ ->	

- 开启多处理器编译，SDL检查：否：
	
- 代码生成->运行库->多线程调试（/mtd)
	
- 语言->符合模式->(否)

输入代码

```c++
#include <torch/torch.h>
#include <iostream>
int main()
{
    torch::Tensor tensor = torch::rand({ 9,9 });
    std::cout << tensor << std::endl;
    return 0;
}
```

生成解决方案并调试

### 如果报缺少VCRUNTIME140_1D.dll

导致程序无法运行就访问下面这个网址下载对应的dll文件

```http
https://www.dll-files.com/
```

复制到C:\windows\system32 和C:\windows\sysWOW64 文件夹下



## cmake+vs配置

### 下载cmake

### 编写CMakelists 

新建一个文件夹，终端输入

```powershell
touch CMakelists.txt
touch main.cpp
mkdir build
```

打开CMakelists,输入

```cmake
cmake_minimum_required(VERSION 3.18)#cmake版本
project(libtorch_test1)#文件夹名字

list(APPEND CMAKE_PREFIX_PATH "./libtorch")#libtorch路径

find_package(Torch REQUIRED)

set(CMAKE_CXX_STANDARD 11)

add_executable(libtorch_test1 main.cpp)#文件夹与cpp

target_link_libraries(libtorch_test1 "${TORCH_LIBRARIES}")
set_property(TARGET libtorch_test1 PROPERTY CXX_STANDARD 11)
```

打开main，输入

```c++
#include <torch/torch.h>
#include <iostream>
int main()
{
	torch::Tensor tensor = torch::rand({ 9,9 });
	std::cout << tensor << std::endl;
    return 0;
}
```

### 调试

终端输入

```powershell
cd build
cmake -G"visual studio 15 2017 Win64" ..#根据自己的vs编译器更改
```

之后会在build文件夹中生成sln文件，之后在用vs打开该sln

项目->设置为启动项目

之后调试，如果报错缺少dll，就去libtorch/lib中把dll复制到build/debug文件夹中

如果报缺少VCRUNTIME140_1D.dll，和windows下部署的解决方法一样