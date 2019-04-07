---
layout:     post
title:      "Anaconda 常用conda命令"
date:       2019-03-24
author:     "董国珍"
header-img: "img/post-bg-2015.jpg"
tags:
    - python
    - Anaconda
---


现阶段我们在学习python时，一般都会被优先推荐使用Anaconda，因为它的环境配置和包的安装快捷简单。

## 什么是Anaconda
首先，我们来认识一下Anaconda：

> Anaconda指的是一个开源的Python发行版本，其包含了conda、Python等180多个科学包及其依赖项。它的包管理比较简单方便.管理虚拟环境也很方便好用。不过也正是因为包含了大量的科学包，所以Anaconda 的下载文件比较大（这并不是什么大问题）。
>
> https://conda.io/docs/user-guide/tasks/index.html 这是官方的帮助文件，是英文版，很全面。

下面我们主要介绍一下部分常用conda命令：



> conda管理：
> - 查看conda版本：conda –version 或者 conda –V
> - 查看更多信息：conda info
> - 更新conda：conda update conda

> 包管理：
> - 搜索包：conda search numpy （在所有添加的仓库以及默认仓库中搜索包 numpy）  
> - 安装包：
> > conda install scipy （向当前环境安装包 scipy）  
> > conda install scipy=0.15.0 （向当前环境安装包 scipy 并指定版本）  
> > conda install scipy curl （向当前环境安装多个包,并指定包版本）  
> - 查看所有已安装包：conda list  
> - 装非conda包/conda中使用pip：  
> > conda install pip （可以使用如下命令安装pip到当前虚拟环境）  
> > pip install numpy （使用pip安装numpy）  
> - 更新：  
> > conda update unmpy （更新指定包）  
> > conda update （更新所有包）   
> > conda update python （更新python）  
> > conda update conda （更新conda本身）  
> - 删除包：  
> > conda remove scipy （删除当前环境中的scipy包）  
> > conda remove scipy curl （删除当前环境中的多个包）  
> > conda remove -n myenv scipy （删除指定环境中的指定包）  




## 在任意文件夹中运行 Jupyter notebook
Jupyter notebook 也是Anaconda 中很好用的一部分，但在使用时，它会默认打开C盘user目录下的文件夹作为home路径，下面再介绍如何使用conda命令在任意文件夹中打开jupyter notebook：（以进入D盘boring文件夹为例）

- 打开Anaconda prompt窗口  
- 输入cd.. (回车)  
- 再输入d: (回车)  
- 再输入cd boring(回车)  
- 再输入：jupyter notebook(回车)  
会用默认浏览器进入jupyter notebook首页
或者可以复制窗口中提示的网址到任意浏览器中打开。


## 总结

1. conda管理
2. 包管理
3. 在任意文件夹中运行 Jupyter notebook

## 参考资料

1. [anaconda （开源的Python包管理器）- 百度百科](https://baike.baidu.com/item/anaconda/20407441?fr=aladdin)
2. [Anaconda - conda 常用命令 - tuzixini](https://blog.csdn.net/tuzixini/article/details/81560980)
