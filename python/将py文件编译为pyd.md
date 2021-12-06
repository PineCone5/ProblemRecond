# 由.py文件转换为.pyd文件
## 概述
将`.py`文件转换为`.pyd`的主要目的是将`.py`文件隐藏起来。  
python支持的文件包括`.py`、`.pyc`、`.pyw`、`.pyo`、`.pyd`，以下简述`.py`与`.pyd`的区别
- `.py`文件
    以 `.py` 作扩展名的文件是Python源代码文件，由`python.exe`解释，可在控制台下运行。  
    当然，也可用文本编辑器或其它专用Python IDE (集成开发环境) 工具进行修改。
- `.pyd`文件
    `.pyd` 文件是由非 Python，其它编程语言编写 (或直接把`.py`文件转换成`.c`中间文件) 编译生成的 Python 扩展模块，是类似`.so`、`.dll` 动态链接库的一种 Python 文件。  

## 先决条件
- 安装了anaconda

## 操作流程
### （一）安装环境
1. 打开**anaconda命令提示符**(Anaconda Prompt)，在其中创建python的虚拟环境，执行以下命令：
    ```bash
    (base) C:\Users\admin>conda conda create -n scade python==3.4.5
    ```
    注：这里使用python3.4.5是因为SCADE 2019R2中python的解释器版本为3.4.5（查看SCADE的python解释器版本的方法见文末）。  
2. 进入创建的环境，并在环境中安装`Cython`库，执行以下命令：
    ```bash
    (base) C:\Users\admin>conda activate scade
    (scade) C:\Users\admin>pip install Cython
    ```
### （二）编写代码  
- 创建`setup.py`文件，并在文件中编写以下代码。    
    ```python
    from setuptools import setup
    from Cython.Build import cythonize
    setup(name="checkrule", ext_modules=cythonize('checkrule.py'))
    ``` 
    注：**name**的值可以随便填写；**ext_modules**的值必须为`cythonize('checkrule.py'))`，其中`checkrule.py`为要转换的`.py`文件。  

### （三）生成pyd文件
1. 在**anaconda命令提示符**(Anaconda Prompt)中切换工作路径到`setup.py`文件所在路径(`setup.py`文件和`checkrule.py`文件必须在同一个路径下)。  
    ```bash
    (scade) C:\Users\admin>cd D:\Test
    (scade) C:\Users\admin>D:
    ```
2. 执行以下命令：  
    ```bash
    (scade) D:\Test>python setup.py build_ext --inplace
    ```
3. 命令执行完成后会生成`.pyd`文件和一些临时文件(临时文件可以删除，不影响`.pyd`文件的调用)。  
<img src="image\文件.png" width="25%">
<img src="image\生成pyd.png" width="70%">  

4. 将`.pyd`文件移动到相应位置  

## 遇到的问题及解决方法
---
### 问题1
- 问题描述  
    无法创建指定python版本的conda环境
- 原因分析  
    conda中原有channel支持的python版本较少，没有指定的python版本安装包
- 解决方法  
    更改conda的channel(换源)，执行以下的命令：
    ```
    conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
    conda config --set show_channel_urls yes
    ```
    执行完命令后再重新执行创建conda环境的命令。
---
### 问题2
- 问题描述  
    在执行`python setup.py build_ext --inplace`时出现以下错误`Unable to find vcvarsall.bat`
- 原因分析  
    缺少VS编译文件`vcvarsall.bat`。
- 解决方法  
    网络上大多要求安装[Visual Studio](https://devblogs.microsoft.com/python/unable-to-find-vcvarsall-bat/#comments)（vcvarsall.bat是Visual Studio的一部分）。
    实际上只需使用anaconda的命令安装libpython即可
        ```bash
        conda install libpython
        ```
---
### 问题3
- 问题描述  
    在执行`python setup.py build_ext --inplace`时出现以下错误  
    ```
    File "C:\Program Files\Python39\lib\distutils\cygwinccompiler.py", line 128, in __init__  
    if self.ld_version >= "2.10.90":  
    TypeError: '>=' not supported between instances of 'NoneType' and 'str'
    ```
- 原因分析  
    执行`ld -v`命令时没有返回结果，导致这个的原因可能是系统变量中没有包含`ld`命令。
- 解决方法    
    添加系统环境变量。
    为系统变量`Path`添加新值`C:\Program Files\ANSYS Inc\v194\SCADE\contrib\Msys64\mingw64\bin\`（此路径为SCADE自带的`mingw64`的`bin`路径）
---
### 问题4
- 问题描述  
    在执行`python setup.py build_ext --inplace`时出现以下错误
    ```
    checkrule.c:213:41: warning: division by zero [-Wdiv-by-zero]
     enum { __pyx_check_sizeof_voidp = 1 / (int)(SIZEOF_VOID_P == sizeof(void*)) };
                                         ^
    checkrule.c:213:12: error: enumerator value for '__pyx_check_sizeof_voidp' is not an integer constant
     enum { __pyx_check_sizeof_voidp = 1 / (int)(SIZEOF_VOID_P == sizeof(void*)) };
    ```
- 原因分析  
    未知。
- 解决方法  
    添加参数`-DMS_WIN64`可解决问题
    ```bash
    python setup.py build_ext --inplace -DMS_WIN64 
    ```
---
### 问题5
- 问题描述  

- 原因分析  

- 解决方法  

---

## 附录
### 附录1：查看SCADE的Python版本
1. 打开SCADE的`ScriptWizard`；  
    <img src="image\ScriptWizard.png" width="30%">
2. 随意选择一个脚本类型(如：`project`)；  
    <img src="image\ScriptWizard1.png" width="20%">
3. 选择创建Python脚本；  
    <img src="image\ScriptWizard2.png" width="80%">
4. 删除脚本中原有代码，编写以下代码；  
    ```python
    import scade
    import sys
    scade.output(sys.version)
    ```
5. 按<kbd>Ctrl</kbd>+<kbd>F5</kbd>键运行脚本，在*Output*中名为*script*的Tab页可以看到python的版本。  
    <img src="image\Output_Script.png" width="70%">

