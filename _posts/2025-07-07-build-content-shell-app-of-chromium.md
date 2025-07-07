---   
title: 构建 chromium 的例子程序备忘
date: 2025-07-07T00:30:38+00:00    
---   

### 下载源代码

```sh 

git clone https://chromium.googlesource.com/chromium/tools/depot_tools.git

export PATH="$PATH:/path/to/depot_tools"

mkdir chromium && cd chromium

fetch --no-history chromium 

# caffeinate fetch --no-history chromium 也可以，防电脑休眠，但没必要
# git fetch --unshallow 下载所有提交记录

```


### 准备构建

```sh

ls `xcode-select -p`/Platforms/MacOSX.platform/Developer/SDKs

cd src

gn gen out/Default

# cat out/Default/build.ninja 看所有可能的编译目标，下面编译的是 Content Shell
# gn args --list out/Default 查看所有可能的编译选项
# gn gen out/gn --ide=xcode 生成 Xcode 项目

```


### 修改编译选项

```sh

# 修改文件 out/Default/args.gn 以变更编译选项

# 几个常改的选项 

is_debug = false # 编译非 Debug 版本，编译会变快
symbol_level = 0 # 不保留符号，编译会变快
is_component_build = false  # 构建成一个大的可执行文件，而不是一堆碎的
enable_nacl = false # 不支持 NACL 插件
use_aura = false
enable_remoting = false # 不支持远程控制

```


### 构建

```sh

autoninja -C out/Default content_shell_app

open ./out/Default/Content Shell.app/

```

