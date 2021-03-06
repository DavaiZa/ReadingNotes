#在没有GUI的Linux上如何可视化Python程序?

------

> 一开始我尝试使用VNC, 然而多次配置都失败了, 总是黑屏. 后来询问实验室一个很厉害的学长, 才知道Jupyter Notebook这个东西.

# 安装Jupyter Notebook

环境: CentOS 7.4

安装过程仿照官方文档[Installing Jupyter Notebook](https://jupyter.readthedocs.io/en/latest/install.html), 我已经安装好了Anaconda, 因此写文章时已经自带该模块. 不过我还是多事, 又升级了一下:

```bash
conda install jupyter
```

# 开启8888端口

我怕多人同时使用jupyter notebook发生冲突, 于是开启了8888-8900网段

```bash
sudo su
firewall-cmd --zone=public --add-port=8888-8900/tcp --permanent
firewall-cmd --reload
exit
```

# 启动Notebook Server

按照官方文档[Running the Notebook](https://jupyter.readthedocs.io/en/latest/running.html#running):

```bash
jupyter notebook
```

会导致notebook只能在127.0.0.1访问到. 为了达成我的目的, 应当使用

```bash
jupyter notebook --ip=0.0.0.0
```

这样就能在外网访问到服务器上的python代码了. 效果相当于把命令的运行目录发布成一个ftp资源, 可以在外网访问到, 并且能被外网浏览器编辑.

官方文档[Running the Notebook](https://jupyter.readthedocs.io/en/latest/running.html#running)还涉及到ssh加密传输等安全细节, 我目前还用不到, 先在此备注一下.

# 在Notebook Server中运行代码

首先创建一个Notebook.

接下来在Notebook中写

```python
%run xxxxx.py
```

运行该notebook, 就能执行xxxxx.py脚本. 这样就不犯愁如何在没有GUI的Linux上显示imshow的图片啦!