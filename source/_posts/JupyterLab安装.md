---
title: JupyterLab安装
date: 2020-05-01 15:50:47
categories: Env
tags: [Jupyter, Linux]
---



JupyterLab 是一个交互式的开发环境，是 Jupyter notebook 的下一代产品，集成了更多的功能，十分好用。

<!--more-->



# 安装Miniconda

1、进入清华大学开源软件镜像站找到Miniconda: https://mirrors.tuna.tsinghua.edu.cn/anaconda/miniconda/

![](http://images.yingwai.top/picgo/jupyterinstallf1.png)

2、找到Miniconda的Linux版本，右键复制链接地址，然后在服务器中下载：

```shell
$ wget -c https://mirrors.tuna.tsinghua.edu.cn/anaconda/miniconda/Miniconda3-py37_4.8.2-Linux-x86_64.sh
```

3、安装刚刚下载的Miniconda：

```shell
$ bash Miniconda3-py37_4.8.2-Linux-x86_64.sh
```

根据提示按enter键或输入yes即可。

4、安装成功后，会在当前用户目录下生成一个miniconda3文件夹。



## 安装pip

在终端中输入以下命令安装pip：

```shell
$ conda install pip
```



### 添加清华源

如果下载速度太慢，可以将conda默认的软件源更换为国内的清华源：

```shell
$ conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
$ conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main/
$ conda config --set show_channel_urls yes

```

添加完成后可以使用`conda info`命令查看是否添加成功。



# 安装并配置JupyterLab

准备工作完成后，就可以开始安装Jupyterlab。

## 安装JupyterLab

在终端中输入：

```shell
$ pip install jupyterlab
```



## 远程访问配置

在终端中打开ipython：

```shell
$ ipython

In [1]: from notebook.auth import passwd

In [2]: passwd()
Enter password:		# 输入你要设置的密码
Verify password:
Out[2]: 'xxxxx...'
```

这里输出的字符串要复制下来。

执行完上面的命令后，退出ipython，然后在终端中输入：

```shell
$ jupyter lab --generate-config
```

修改配置文件：

```shell
$ vi .jupyter/jupyter_notebook_config.py
```

更改内容如下：

```shell
# 将ip设置为*，意味允许任何IP访问
c.NotebookApp.ip = '*'
# 这里的密码就是上面生成的那一串
c.NotebookApp.password = 'xxxxx...' 
# 服务器上并没有浏览器可以供Jupyter打开 
c.NotebookApp.open_browser = False 
# 监听端口设置为8888或其他自己喜欢的端口 
c.NotebookApp.port = 8888
# 允许远程访问 
c.NotebookApp.allow_remote_access = True
```

启动jupyter服务：

```shell
$ jupyter lab --allow-root
```

此时在浏览器搜索框中输入`你的服务器ip:你设置的端口`，然后在打开的页面中输入密码就可以进入jupyterlab：

![](http://images.yingwai.top/picgo/jupyterinstallf2.png)



## 后台运行JupyterLab程序

JupyterLab启动后占用了一个终端窗口，可以用`nohup`命令使JupyterLab在后台运行，并且关闭当前终端也不会停止运行：

```shell
$ nohup jupyter lab &
```

* `nohup`命令

  用途：Run COMMAND, ignoring hangup signals.

  输出文件：程序的输出默认重定向到当前文件夹下的`nohup.out`文件中。也可以通过`nohup COMMAND > FILE`命令的方式将输出文件重定位到指定的`FILE`文件中。如果要查看JupyterLab的日志文件，可以打开`nohup.out`文件进行查看。

* `&`命令

  作用：在后台运行程序



## 查看、关闭后台运行进程

`job -l`命令查看当前终端中后台运行的进程，如果关闭终端后不能显示，需要使用`ps`命令。

`ps -aux | grep jupyter`查看运行的`jupyter`进程：

![](http://images.yingwai.top/picgo/jupyterinstallf3.png)

用户名后面的数字就是JupyterLab的pid，使用`kill -9 pid`命令关闭运行中的JupyterLab。