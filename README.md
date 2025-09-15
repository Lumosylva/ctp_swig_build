# ctp_swig_build
**一句话介绍项目**：一键自动编译CTP C++接口为Python接口。

## 1. 前言

目前上期技术CTP接口提供的API版本是 C++版本，本文主要介绍在Windows 64位平台下利用Swig工具将CTP C++接口转换为Python可调用的接口。

## 2. 准备工作

- **下载 CTP API 压缩包**

从 SimNow 官网PC标签页下载CTP API 压缩包，注意非交易时间段此网站不能访问。这里以`v6.7.8`版本为例（你可以自行用其他版本，步骤一样），64位的API文件包解压后清单如下：

```reStructuredText
error.dtd
error.xml
ThostFtdcMdApi.h
ThostFtdcTraderApi.h
ThostFtdcUserApiDataType.h
ThostFtdcUserApiStruct.h
thostmduserapi_se.dll
thostmduserapi_se.lib
thosttraderapi_se.dll
thosttraderapi_se.lib
```

- **安装 Swig**

  本文中所用的Swig是 **`swigwin-4.3.0`** 版本，[点击此处下载](https://zenlayer.dl.sourceforge.net/project/swig/swigwin/swigwin-4.3.0/swigwin-4.3.0.zip?viasf=1)，更多Swig版本[下载地址](https://sourceforge.net/projects/swig/files/swigwin/)。

- **安装 Python**

  推荐使用`UV`来安装，下面有使用说明。也可以使用其他 Python 管理工具，但是需自行配置相关环境。注意要安装64位版本，将环境变量配置好。本文所用的是 **`3.13.6`** 版本，如果自用到别的版本，下列步骤一致。

- **安装 Visual Studio**

  主要是用到其中的 `MSVC` 和 `Ninja`，本文所用的是**Visual Studio 2022**，注意安装Visual Studio的时候勾选上C++开发。

## 3. 安装UV和Python环境

本项目推荐使用 `UV` 来管理 Python 安装和依赖包安装

1. 安装UV

   On Windows

   **方式一：全局安装(推荐方式，二选一)**

   在PowerShell中运行下述命令(注意不是cmd)

   ```bash
   powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
   ```

   **方式二：单独在 Python 环境中安装(二选一)**

   ```bash
   pip install uv
   ```

   On Linux

   ```bash
   curl -LsSf https://astral.sh/uv/install.sh | sh
   ```

3. 安装 Python(方式一进行这一步，方式二直接跳过)，我自己用的是 3.13.6，你可以安装自己需要的版本

   ```bash
   uv python install 3.13
   ```

4. 在项目根目录运行下述命令将 Python 虚拟环境安装到项目根目录(与上一步全局安装的 Python 隔离)

   ```bash
   uv venv --python 3.13 .venv
   ```


## 4. 使用

项目使用 `SWIG + MSVC + Meson + Stubgen` 的组合来编译CTP C++ API生成Python扩展模块。

### 主要功能：

meson.build文件：

- 配置了C++17编译环境

- 自动查找Python解释器和SWIG工具

- 为行情API（thostmduserapi）和交易API（thosttraderapi）分别配置SWIG包装代码生成

- 设置了正确的包含目录和库文件链接

- 自动安装生成的Python文件和DLL文件


build.py文件：

- 检查所有必要的依赖项（SWIG、Meson、Ninja）

- 自动设置和清理构建目录

- 配置Meson构建（支持MSVC环境）

- 执行编译和安装过程

- 使用mypy自带的stubgen生成类型存根文件

- 提供了多种命令行选项（仅配置、跳过存根生成等）

### 使用方法：

1. 激活 Python 虚拟环境：

   ```bash
   .venv\Scripts\activate
   ```

2. 运行构建：

   ```bash
   python build.py
   ```

3. 测试接口：

   demo文件为 `ctp_demo.py`，运行即可。


### 主要特点：

- ✅ 支持多线程（-threads参数）
- ✅ 自动处理中文编码转换
- ✅ 生成类型存根文件提供IDE支持
- ✅ 支持Windows MSVC编译环境
- ✅ 自动复制必要的文件
- ✅ 无需打开Visual Studio即可实现一键编译

构建脚本将会自动：

- 编译生成pyd文件
- 复制到项目根目录的ctp文件夹
- 自动重命名，在文件名前添加下划线
- 同时处理相关的.lib文件

这样可以确保SWIG生成的Python模块能够正确找到并导入底层的C扩展模块，构建完成后，将得到完整的Python扩展模块，可以直接在Python代码中使用CTP API的所有功能。

## 5. 项目结构

```reStructuredText
ctp/
├── _thostmduserapi.cp313-win_amd64.pyd		# 重命名后的行情API模块
├── _thostmduserapi.cp313-win_amd64.lib		# 重命名后的库文件
├── _thosttraderapi.cp313-win_amd64.pyd		# 重命名后的交易API模块
├── _thosttraderapi.cp313-win_amd64.lib		# 重命名后的库文件
├── thostmduserapi.i		# 接口文件，用于告诉swig为哪些类和方法创建接口。
├── thosttraderapi.i		# 接口文件，用于告诉swig为哪些类和方法创建接口。
├── thostmduserapi.py		# SWIG生成的Python接口
├── thosttraderapi.py		# SWIG生成的Python接口
├── thostmduserapi.pyi		# 利用mypy自带的stubgen生成类型存根文件
├── thosttraderapi.pyi		# 利用mypy自带的stubgen生成类型存根文件
├── __init__.py				# Python包初始化文件
├── thostmduserapi_se.dll	# 行情API动态库
├── thosttraderapi_se.dll	# 交易API动态库
└── ...其他文件
```

## 6. 后续工作

**提示 import \_\_builtin\_\_ 错误**

当你打开 `thostmduserapi.py` 或 `thosttraderapi.py` 时，可能会出现以下错误

![thostmduserapi_error](/assets/thostmduserapi_error.png)

只需改为以下代码即可解决：

![thostmduserapi_no_error](/assets/thostmduserapi_no_error.png)

