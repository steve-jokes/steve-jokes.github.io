---
title: Qt(C++) 跨平台集成 Python(with site-package) 完全指南
date: 2017-03-14 10:43:46
tags:
  - qt
  - c++ 
  - python
  - 跨平台
  - 部署
categories:
  - 技术
  - Qt
  - 部署
---

Qt 无疑是桌面客户端的开发的上上之选之一. 跨多平台自然不必多说, 更新稳定, 工具支持完整, 社区健全. 能够让开发者保持一个比较高的开发效率. 如果你还嫌业务逻辑的开发效率不够高, 或者移植性不够好. 没有问题, 你还可以选择 Python 脚本来实现你的业务逻辑. 将 Python 集成到你的 Qt 应用中. 这样你可以同时高效地开发 GUI, 高效地编写业务逻辑. 这个技术栈我已经使用了两年, 虽然现在有些想法觉得可能有更好的选择(比如 Qt(QML) + Golang), 但是这套搭配我还是非常推崇的.

但是在使用这个搭配的过程中, 有一个问题困扰了我很久. 就是 Python 环境如何干净地集成到 Qt 中, 既不影响用户自身的 Python 环境, 也不要受到用户本地 Python 的影响. 由于本地环境的干扰, 很长一段时间内我发布的程序都是在使用用户的 Python 环境, 直到用户低版本的 Python 出现了 bug 才发现了这个问题. 而 Python 环境集成的错误, 也导致我不能够正常的使用我需要的 site-package, 一番摸索之后, 这个问题总算是解决了. 最近有点时间, 对之前的处理做个总结, 免得后人踩坑.

<!--more-->
![]()

# 本文适应人群
作者使用的是 Python 官方使用的 CPython 解释器, PyPy 不熟悉, 不在此讨论范围内. 说是集成完全指南, 实际上我会略过集成这一部分, 主要讲环境的部署. 集成我会给出传送门, 请大家自行了解. 本文尤其适合: 查遍互联网关于 C++ python 部署相关资料, 还是不能成功部署独立 python 环境的开发者. 因为**Qt 本身带来了一些坑**

# Python 集成
作者使用的 Python 自带的集成方式 [Embedding Python in C++](https://docs.python.org/2.7/extending/embedding.html#embedding-python-in-c), 虽然原始, 但是比较灵活可用. 实际在社区中推荐下来 [CFFI](http://cffi.readthedocs.io/en/latest/index.html) 是比较现代的 C/Python 交互方式. 这里需要注意的是链接方面的相关问题, Qt 使用 qmake 来生成 makefile, 所以你要在 pro 文件中使用 Qt 的语法来写你的库依赖.

# 部署环境
## Mac
### 独立 Framework 制作
我使用的是 brew install 的 python, 当前版本 2.7.13, 它的框架在 `/usr/local/Cellar/python/2.7.13/Frameworks/` 下, 把这个框架拷出来就可以放到工程里来依赖, 链接, 部署了. 框架下的软连接是方便别人引用用的, 基本都可以删掉, 当然你留着也没什么问题. 我们最需要的是 `/Versions/2.7/` 下的 `include` 和 `lib` 两个目录.

做到这里就可以了, 但是如果你用 pip install 过很多 site package, 你的框架可能非常大, 这个时候需要用 virtualenv 来制作一个比较精简的框架(新手或不在乎请跳过) 这部分我在自己的开发文档中写过, 不想重写一遍, 直接贴了:

> 
#### How to get a tiny python framework
Our own system python or brew python always full of site-package that maybe huge and redundant. when embedding python framework for c++, we want it to be small and clean. so how can we get a tiny python.
>
##### virtualenv
First we want to get a clean python environment without any influence to  our current python. that's what virtualenv is doing:
- pip install -U virtualenv (if necessary)
- virtualenv --no-site-packages tiny_python (no site package from outside)
- pip install your_necessary_site_package
>
##### remove the link
now we have a tiny_python, it can be used as a framework to a Qt program. but before that, there is one more stop to go:
in tiny_python stays some soft link to origin python. it's good for virtualenv to create a new python env, but not good for deploy. we should copy all the origin files to replace with the link, so we can ship the code.
>
##### make it smaller
we notice the tiny_python now have lots of py and pyc. which can be optimized. use `compileall.compile_dir()` to compile all the py to pyc, and delete py.
>
##### framework making
So far I'm not familiar with framework making. but that alright, i fount a workarount:
>
- copy Python.Framework from origin python. if your user brew on mac, it's in /usr/local/Cellar/python/2.7.13/Frameworks
- replace the old big folder with our lite version
>
##### notice
because we use virtualenv, something have been changed by virtualenv. like site, you can't use `set.getsitepackages()` any more.

### 编译依赖
Mac Python.Framework 单独剥离出来后就可以存放到自己的工程中提供引用了, 我在 pro 文件中用 **PYTHON_FRAMEWORK_DIR** 来表示其路径.
```
QMAKE_CFLAGS += -I$$PYTHON_FRAMEWORK_DIR/Versions/2.7/include/python2.7
QMAKE_LFLAGS += -L$$PYTHON_FRAMEWORK_DIR/Versions/2.7/lib/python2.7/config -lpython2.7 -ldl -framework CoreFoundation

INCLUDEPATH += $$PYTHON_FRAMEWORK_DIR/Versions/2.7/include/python2.7
DEPENDPATH += $$PYTHON_FRAMEWORK_DIR/Versions/2.7/lib/python2.7/config
LIBS += -L$$PYTHON_FRAMEWORK_DIR/Versions/2.7/lib/python2.7/config -lpython2.7
```
至此你的 Qt 程序应该能够使用 Python.h 的功能, 在 Debug 环境中顺利调用 Python 解释器, 来执行你的 python 脚本, 使用你指定的 site-pacakge 了.

### 部署
Qt 的部署通常通过 macdeployqt 命令来完成, 它会自动查找你的应用使用到哪些 Framework, 并将其复制到 .app 文件的 Bundle 下一个叫做 Frameworks 的目录中. 
但是对于 Python.Framework 却不能正确处理, 需要我们自己手动复制, 并调整可执行程序对库的查找路径, 所以我在执行 macdeployqt 之前会做这么一个处理:
```
mkdir $YOUR_APP_PATH/Contents/Frameworks/
mkdir $YOUR_APP_PATH/Contents/Frameworks/Python.framework/
cp -r $$PYTHON_FRAMEWORK_DIR/ $YOUR_APP_PATH/Contents/Frameworks/Python.framework/
install_name_tool -change /usr/local/opt/python/Frameworks/Python.framework/Versions/2.7/Python @executable_path/../Frameworks/Python.framework/Versions/2.7/Python $YOUR_APP_PATH/Contents/MacOS/YOUR_APP_NAME)
```

这样, 你的 Mac 程序应该就可以独立的发布出去了.

## Windows
我在 win7 下制作的包, 按道理 win10 应该不会有太大的差别.

### 独立 Framework 制作
### 编译依赖
### 部署
