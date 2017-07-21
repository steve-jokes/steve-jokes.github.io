---
title: Qt 集成 ImageMagick 实录
tags:
  - qt
  - c++
  - ImageMagick
  - 部署
categories:
  - 技术
  - Qt
  - 部署
date: 2017-07-19 16:47:17
---

近来需要基于 Qt 写一个桌面客户端视频编辑工具, 可以切割视频分段, 为部分分段增加蒙版图片, 再生成一份配置以供后续使用. 这期间使用到了 FFmpeg 和 ImageMagick 两个优秀的第三方开源框架. FFmpeg 的集成最终使用了调用命令行参数的方式, 因为 FFmpeg 提供的 C 形式 api 难于阅读, 而且没有一份详尽的 capi 文档, 故作罢. 而 ImageMagick 的集成并没有使用命令行方式, 原因是 ImageMagick 似乎依赖了比较多的东西, 调用命令行时会遇到诸多问题. 比如腾讯退出的图片压缩工具智图, 就需要用户额外自己安装一个 ImageMagick 以供调用. 还提供了一个 GUI 的 ImageMagick 安装方式, 使用起来体验并不理想. 所以最终我选择了使用 ImageMagick 的 C++ 形式来集成之. 但是这个过程也是一波三折, 记录一下, 后来人有缘相见, 免得白折了诸多时间. 

<!--more-->

![qt and ImageMagick](wizard.png)

# 获得 ImageMagick
最初我使用 home brew 自动安装的版本来使用, 可是部署过程中遇到了种种问题, 在 IRC 频道和 Github 上多方交流都未能解决. 虽然现在这些问题看起来都没什么了, 不过这里我还是建议大家自己获取[源码](https://github.com/ImageMagick/ImageMagick)来编译, 因为 ./configure 过程中有一些参数需要配置的, 使用 brew 指定的话, 我目前还没有亲测过.

这里放送上我的操作步骤:
```
git clone git@github.com:ImageMagick/ImageMagick.git
cd ImageMagick
./configure --prefix=/path/to/install/dir/ImageMagickInstall --without-x --without-webp --without-xml --without-tiff --without-jpeg --with-png  --enable-shared=yes --enable-static=yes --enable-zero-configuration
make
make install
```
注意几个地方:
- 推荐使用 --prefix 重新定义安装路径, 不要加到系统默认路径中, 这样之后也比较容易在本机测试程序在无 ImageMagick 环境下能否正常运行
- 根据需要自行选择需要 --without-哪些模块
- 推荐使用 --enable-zero-configuration 参数, 这样程序不会依赖 etc 中的 xml 配置文件, 在后期部署的时候, 能够省去不少麻烦
- 这样安装结束后, 在 ImageMagickInstall 目录中, 会生成 Include 目录, lib 目录. lib 目录中有动态库和静态库, 你可以自行选择通过哪种方式来集成
- lib 中有几个链接可以使用, 最好不要使用哦, 因为链接写死了指向的绝对路径, 不方便你将动态库移到其他目录

# 配置 pro 文件
之后就可以在 Qt.pro 文件中配置对 ImageMagick 的引用啦:

```
mac {
    DEFINES += "MAGICKCORE_HDRI_ENABLE=1"
    DEFINES += "MAGICKCORE_QUANTUM_DEPTH=16"

    INCLUDEPATH += $$PWD/third_tools/mac/image_magick/include/ImageMagick-7 
    DEPENDPATH += $$PWD/third_tools/mac/image_magick/include/ImageMagick-7

    LIBS += -L$$PWD/third_tools/mac/image_magick/lib -lMagick++-7.Q16HDRI.3 -lMagickWand-7.Q16HDRI.3 -lMagickCore-7.Q16HDRI.3
}
```
注意几个地方:
- 这里我写的是我放置 include lib 目录的路径, 你需要自行修改成你的目录
- 添加库的时候, 建议不要使用 -lMagick++-7.Q16HDRI 而是 -lMagick++-7.Q16HDRI.3, 原因是前者是个链接, 可能记录到原始地址去了
- 两个宏定义是我之前根据 Magick++-config --cppflags --cxxflags --ldflags --libs 得到的, 照着用就可以

# 引用, 编译
qmake 之后, 你就可以在程序中顺利的 `#include <Magick++.h>` 并且编译链接成功啦, 不出意外的话, 现在应该可能正常使用这些功能

# 部署
当发布 release 版本时, 需要为程序附加上使用到的动态库, 要用到 qt 提供的 macdeployqt 命令 
`macdeployqt path/to/your/app -qmldir=path/to/your/qmldir`
当然如果你写的是纯 cpp 代码, 不需要添加 -qmldir 参数

这期间你会遇到这样一个问题:  如果你像我一样, 移动过动态库的目录的话, macdeployqt 提示找不到 ImageMagick 的动态库, 你会发现打印出来的动态库地址是你一开始编译生成时动态库的地址
这是因为动态库的 id 记录了它的原始地址, macdeployqt 会根据这个地址去复制动态库到程序的 `Contents/Frameworks` 目录下, 你可以使用 `otool -L dylib/path` 来查看它的 id 地址. 就是第一行你看到的路径. 遇到这个问题不要慌, 苹果提供了 `install_name_tool` 工具来修复这个问题. 简单暴力的做法就是 `install_name_tool -id dylib/absolute/path dylib/absolute/path`, 再次 otool 你会发现第一行的路径已经修复, 可是还有一些依赖的路径仍然是旧的绝对路径, 这时候要使用 `install_name_tool -change old/absolute/dependence/lib/path new/absolute/dependence/lib/path target/absolute/lib/path` 来修复.
完成了这些, 再次使用 `macdeployqt` 来部署动态库时, 这些动态库就能顺利地被复制到 `Contents/Frameworks` 目录下, 并且处理好依赖路径的更新了

# 未完成
使用这种方式部署以后, 在部分机器上还会有动态库找不到而打不开的问题. 这是因为有些机器没有安装 libtool, 而我并没有将其编译依赖下去. 有一个临时的解决办法是: [https://coderwall.com/p/8pajlg/fixing-library-not-loaded-usr-local-lib-libltdl-7-dylib](https://coderwall.com/p/8pajlg/fixing-library-not-loaded-usr-local-lib-libltdl-7-dylib), 做到这里, 我已经可以在一个陌生环境中运行我的代码, 并且使用 ImageMagick 了. 以上.



