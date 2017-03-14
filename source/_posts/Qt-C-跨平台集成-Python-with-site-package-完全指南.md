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

不过在使用这个搭配的过程中, 有一个问题困扰了我很久. 就是 Python 环境如何干净地集成到 Qt 中, 既不影响用户自身的 Python 环境, 也不要受到用户本地 Python 的影响. 由于本地环境的干扰, 很长一段时间内我发布的程序都是在使用用户的 Python 环境, 而我完全都没有发觉, 直到用户低版本的 Python 出现了 bug 才发现了这个问题. 而 Python 环境集成的错误, 也导致我不能够正常的使用我需要的 site-package, 一番摸索之后, 这个问题总算是解决了. 最近有点时间, 对之前的处理做个总结, 免得后人踩坑.

<!--more-->

![qt with python](qt_with_python.jpg)
>配图来自 https://www.udemy.com/create-simple-gui-applications-with-python-and-qt

# 本文适应人群
---
我使用的是 Python 官方使用的 CPython 解释器, PyPy 不熟悉, 不在此讨论范围内. 说是集成完全指南, 实际上我会略过集成这一部分, 主要讲环境的部署. 集成我会给出传送门, 请大家自行了解. 本文尤其适合: 查遍互联网关于 C++ python 部署相关资料, 还是不能成功部署独立 python 环境的开发者. 因为 **Qt 本身带来了一些坑**

# Python 集成
---
我使用的 Python 自带的集成方式 [Embedding Python in C++](https://docs.python.org/2.7/extending/embedding.html#embedding-python-in-c), 虽然原始, 但是比较灵活可用. 实际在社区中推荐下来 [CFFI](http://cffi.readthedocs.io/en/latest/index.html) 是比较现代的 C/Python 交互方式. 大家可以自行了解, 实验.
这阶段需要注意的是链接方面的相关问题, Qt 使用 qmake 来生成 makefile, 所以你要在 pro 文件中使用 Qt 的语法来写你的库依赖.

# 部署环境
---
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

install_name_tool 可以改变你的可执行文件对依赖框架的查找位置, 这样, 你的 Mac 程序应该就可以独立的发布出去了.

## Windows
我在 win7 下制作的包, 按道理 win10 应该不会有太大的差别.

### 独立 Framework 制作
Qt 集成 Python 的编译和链接过程, 要依赖 Python27 目录下的 include 和 libs 文件夹
而发布(部署)则需要依赖动态库 python27.dll (从 C:\Windows\System32\ 目录下复制). 此外还需要 Python27 目录中的标准库和扩展库 DLLs 和 Lib.
结构如图所示
```
├── Python27/
│   ├── DLLs/
│   └── Lib/
├── include/
├── libs/
└── python27.dll
```
同样的, 理论上你也可以使用 VirtualEnv 来减小 Python27 库的体积. 截止发文时我还没有测试过, 就不啰嗦了.

### 编译依赖
没啥好说的, 就是普通的添加依赖关系
```
QMAKE_CFLAGS += -I$$PYTHON_FRAMEWORK_DIR/include
QMAKE_LFLAGS += -L$$PYTHON_FRAMEWORK_DIR/libs -lpython2.7

INCLUDEPATH += $$PYTHON_FRAMEWORK_DIR/include
DEPENDPATH += $$PYTHON_FRAMEWORK_DIR/libs

LIBS += -L$$PYTHON_FRAMEWORK_DIR/libs -lpython27
```

### 部署
Python 框架的文件部署, 只需要将上面所示的 python27.dll 和 Python27 文件夹放到可执行文件的同级目录下就行(**注意大小写**).

但是光部署文件可不够用, 程序运行起来以后仍然会访问 C:\Python27 去寻找 Python 运行环境, 我们需要在 Python 初始化之前加一段代码:
```
// 指定使用程序自带的 Python 环境
QString pythonHome = QCoreApplication::applicationDirPath().replace("/","\\")+"\\Python27";
string strPythonHome = pythonHome.toStdString();
Py_SetPythonHome(const_cast<char*>(strPythonHome.c_str()));

// 初始化 Python 解释器实例
if (!Py_IsInitialized()){
    Py_Initialize();
}
```
这里有个坑: 如果你上网搜索 C++ with Python embedded, 查到 Py_SetPythonHome() 这个解决办法, 可是在实际操作的过程中, 仍然没有办法成功指定 PythonHome 的路径. 那是因为 Qt 使用的 string 类型 QString 的干扰. 通常情况下, 我们会选择使用 QString().toLocal8bit().data() 来获得字符串, 这种情况下生成的字符串经测试 Py_SetPythonHome 是无效的. 必须调用 toStdString() 再 c_str() 成 char* 才能正常使用.

做完这些, 你的 Python 程序很大程度上已经可以运行起来了, 但是实际上你的 Python 代码中 import 的 sys 模块是有方法缺失的. 因为 Python 模拟器不是正常调起, 所以 sys.argv 是不存在的. 这会造成你对 sys.argv 的使用出现异常, 也会影响到很多 site-package 的正常引用, 解决的办法是调用 Py_SetProgramName(), 和 PySys_SetArgv() 获得启动参数.

```
//main.cpp

int g_argc;
char **g_argv;

int main(int argc, char *argv[])
{
    // 保存全局进程参数, 留给 Python 模块初始化使用
    g_argc = argc;
    g_argv = argv;
    ...
}

```

```
//python_invoke.cpp

// 初始化 Python 解释器实例
if (!Py_IsInitialized()){
    Py_Initialize();

    // 自带的 Python 环境需要指定进程的 argc 和 argv, 否则无法使用 sys.argv
    Py_SetProgramName(g_argv[0]);
    PySys_SetArgv(g_argc, g_argv);

    // init support for multi threads
    PyEval_InitThreads();

    // executed between running threads, to release global lock locked bt PyEval_InitThreads()
    PyEval_ReleaseThread(PyThreadState_Get());
}

```

做到这一步, 你的程序应该就能够顺利运行了

# 总结
说 Python 是一门胶水语言确实不假, 完成部署以后, 我后知后觉的发现其实 Python 对 C 的粘合其实做得还行. 只不过互联网上大多都是关于如何连接编译调用函数的知识, 对发布的信息比较少. 尤其是到了 Qt 这个领域又有几个小坑要过, 所以看起来像是一团迷雾, 恶心的要死. 我几乎要写一篇 **Qt Python 混合编程从入门到放弃** 了. 不过说到转移技术栈这回事, 最近听身边大神安利 Golang 的部署极为简洁干净, 而且跨平台, 完全没有 Python 这样弯弯绕绕的东西, 心中长草, 下个月要学习一下.
