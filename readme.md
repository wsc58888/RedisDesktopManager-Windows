转载：https://github.com/necan/RedisDesktopManager-Windows
## 前言

因为作者在 0.9.4 版本之后选择对所有的安装包收费，不再提供安装包下载，但是源码依旧公开。

但是网络上关于 Redis Desktop Manager 的编译教程都是 Linux 下的，没有任何参考价值。而官方文档提供的步骤只有寥寥几个字，没有任何可操作性。本人摸索了几个小时，终于摸清编译打包的完整流程。

更新：觉得麻烦的，可以直接下载本人编译打包好的[安装包](https://github.com/wsc58888/RedisDesktopManager-Windows/releases)

 

## 编译过程

### 安装工具

#### 安装 VSCode 2015

到 [http://blog.postcha.com/read/66](http://blog.postcha.com/read/66) 下载 VSCode 专业版，自定义安装，一定要勾选 VC ++，然后一直下一步，视机器配置而定，一般大约一个半小时装完。

#### 安装 Qt 5.9

到 [http://mirrors.ustc.edu.cn/qtproject/archive/qt/5.9/](http://mirrors.ustc.edu.cn/qtproject/archive/qt/5.9/) 下载最新到 Qt 5.9 版本，一直下一步就行，大约半个小时左右。

#### 安装 CMake

到 [https://cmake.org/download/](https://cmake.org/download/) 下载 32 位的版本，安装时注意勾选添加到 PATH

#### 安装 NSIS

安装打包工具 [http://nsis.sourceforge.net/Download](http://nsis.sourceforge.net/Download)

#### 安装 Python 2

到 https://www.python.org/downloads/ 下载安装 Python 2.7

#### 安装 OpenSSL

下载并安装 [Win 32 OpenSSL 1.0.x](https://slproweb.com/products/Win32OpenSSL.html) 版本

### 编译 Redis Desktop Manager

打开 “VS2015 x86 本机工具命令提示符”

#### 获取源码

```powershell
git clone --recursive https://github.com/uglide/RedisDesktopManager.git D:\redisdesktopmanager
cd D:\redisdesktopmanager
```

#### 编译 libssh2

```powershell
cd ./3rdparty/qredisclient/3rdparty/qsshclient/3rdparty/libssh2
cmake -G "Visual Studio 14 2015" -DCRYPTO_BACKEND=OpenSSL -DBUILD_EXAMPLES=off -DBUILD_TESTING=off -H. -Bbuild
cmake --build build --config "Release"
```

#### 设置版本号

版本号自己到 Github 找

```powershell
cd D:\redisdesktopmanager
set VERSION=0.9.4.1055
"D:\Program Files\Python\python27\python.exe" ./build/utils/set_version.py %VERSION% > ./src/version.h
"D:\Program Files\Python\python27\python.exe" ./build/utils/set_version.py %VERSION% > ./3rdparty/crashreporter/src/version.h
```

#### 编译 crashreporter

```powershell
cd ./3rdparty/crashreporter
"D:\Qt\Qt5.9.6\5.9.6\msvc2015\bin\qmake.exe" CONFIG+=release DESTDIR=D:\redisdesktopmanager\bin\windows\release
powershell -Command "(Get-Content Makefile.Release).replace('DEFINES       =','DEFINES       = -DAPP_NAME=\\\"RedisDesktopManager\\\" -DAPP_VERSION=\\\""%VERSION%"\\\" -DCRASH_SERVER_URL=\\\"https://oops.redisdesktop.com/crash-report\\\"')" > Makefile.Release2
nmake -f Makefile.Release2
```

#### Qt 编译

打开 Qt Creator，打开 `./src/rdm.pro`

选择 “Deaktop Qt 5.9.6 MSVC2015 32bit”，构建选择 release，点击构建项目。

### 打包

```powershell
cd D:\redisdesktopmanager
copy /y .\bin\windows\release\rdm.exe .\build\windows\installer\resources\rdm.exe
copy /y .\bin\windows\release\rdm.pdb .\build\windows\installer\resources\rdm.pdb
D:\redisdesktopmanager\3rdparty\gbreakpad\src\tools\windows\binaries\dump_syms .\bin\windows\release\rdm.pdb  > .\build\windows\installer\resources\rdm.sym
cd build/windows/installer/resources/
D:\Qt\Qt5.9.6\5.9.6\msvc2015\bin\windeployqt --no-angle --no-opengl-sw --no-compiler-runtime --no-translations --release --force --qmldir D:\redisdesktopmanager\src\qml rdm.exe
rmdir /S /Q .\platforminputcontexts
rmdir /S /Q .\qmltooling
rmdir /S /Q .\QtGraphicalEffects
del /Q  .\imageformats\qtiff.dll
del /Q  .\imageformats\qwebp.dll
cd D:\redisdesktopmanager
call "C:\\Program Files (x86)\\NSIS\\makensis.exe" /V1 /DVERSION=%VERSION% ./build/windows/installer/installer.nsi
```

打包后的文件：`D:\redisdesktopmanager\build\windows\installer\redis-desktop-manager-0.9.4.1055.exe`

