---
title: jupyterlab配置
layout: post
date: '2020-01-04 19:24:05'
tags: jupyter
color: rgb(255,90,90)
cover: "../assets/imgs/jupyter.png"
subtitle: jupyterlab配置
---

### 目录

* any list
{:toc}

### 简介

`JupyterLab`是一个交互式的开发环境，是`Jupyter notebook`的下一代产品，集成了更多的功能。

### 安装

  1. 安装python环境。
  2. 安装jupyterlab：
```sh
pip install jupyterlab
```
  3. 安装ipython：
```sh
pip install ipython
```

### 配置

  1. 生成配置文件：
```sh
jupyter notebook --generate-config
```
  2. 设置访问密码（比一般教程方便的多）：
```sh
jupyter notebook password
```
  3. 修改配置文件，修改`~/.jupyter/jupyter_notebook_config.py`的以下内容(或者把下面的内容直接复制到文件中)：
```python
c.NotebookApp.allow_remote_access = True  # 允许远程访问
c.NotebookApp.allow_root = True  # 允许root权限启动jupyter
# c.NotebookApp.enable_mathjax = True  # 可选配置，是否使用MathJax渲染，如果网络比较慢就不用开启啦，本地的话没问题
c.NotebookApp.ip = '*'  # 允许所有ip访问
c.NotebookApp.open_browser = False  # 不自动打开浏览器
c.NotebookApp.port = 8888  # 可以手动设置端口
c.ContentsManager.allow_hidden = True  # 是否可以访问隐藏文件
```

### nbextensions插件配置

个人使用比较少，暂时不配置这一项，可以参考其它博客。

### 启动jupyter-lab

在终端输入：

```sh
jupyter-lab
```
