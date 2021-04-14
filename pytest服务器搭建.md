# pytest服务器搭建

## 安装git

安装命令：
```shell script
[root@Testing-Platform ~]# yum install git
```

安装成功之后检查是否安装成功
```shell script
[root@Testing-Platform ~]# git --version
git version 1.8.3.1
```

安装python3.8.2环境
```shell script
mac@MacdeMacBook-Pro  ~  scp /Users/mac/Downloads/Python-3.8.2.tgz root@10.0.15.198:/root/download
root@10.0.15.198's password:
Python-3.8.2.tgz                                                   100%   23MB   6.6MB/s   00:03
```
解析：
此时运行命令应该是在自己的本机环境，而不是ssh服务器上，上传压缩包至服务器上

查看服务器是否已经上传成功
```shell script
[root@Testing-Platform download]# ls
install.sh  jdk-8u171-linux-x64.rpm  Python-3.8.2.tgz
```

解压文件
```shell script
tar -xvzf Python-3.8.2.tgz
```

进入目录
```shell script
cd Python-3.8.2
```

添加配置文件
* 1 配置安装目录，执行命令
```shell script
[root@Testing-Platform Python-3.8.2]# ./configure --prefix=/usr/local/python
```

* 2 应用配置，执行命令
```shell script
[root@Testing-Platform Python-3.8.2]# ./configure
```
* 3 编译源码，执行命令
```shell script
[root@Testing-Platform Python-3.8.2]# make && make install
```

遇到问题
```shell script
You are using pip version 8.1.2, however version 21.0.1 is available.
You should consider upgrading via the 'pip install --upgrade pip' command.
```
解析：
执行命令：： pip install --upgrade pip

gitlab上拉取项目
```shell script
git clone 项目http地址
```
生成requirements.txt依赖
```shell script
pip3 freeze > requirements.txt
```
安装依赖：
```shell script
[root@Testing-Platform homedotest]# pip3 install -r requirements.txt
```

python下载包安装方法：
```shell script
python setup.py install
```
解析：
安装tar.gz包：cd到解压后路径

linux如何复制文件夹和移动文件夹
```shell script
一、文件复制命令cp
如将/test1目录下的file1复制到/test3目录，并将文件名改为file2,可输入以下命令：
cp /test1/file1 /test3/file2

二、文件移动命令mv
如将/test1目录下的file1复制到/test3 目录，并将文件名改为file2,可输入以下命令：
mv /test1/file1 /test3/file2
```


下载allure压缩文件
```shell script
第一步：下载-解压-配置路径
Allure包下载地址：
https://github.com/allure-framework/allure2/releases

下载后，上传到服务器，我放在了/usr/local这个路径下，解压：unzip allure-2.7.0.zip
接下来配置allure的环境变量：vi /etc/profile
在最后追加一句：export PATH=$PATH:/usr/local/allure-2.7.0/bin
保存并退出：wq
加载一下配置文件：source /etc/profile
验证：allure --version

第二步：
生成json格式的临时报告

第三步：
os.system("allure generate ./temp -o ./report --clean")

allure generate 命令，固定的
./temp 临时的json格式报告路径
-o 输出output
./report 生成的allure报告路径
--clean 清空./report的原来报告
```
解析：
因为allure是需要java环境的，并且是jdk1.8及以上版本，所以提前在服务器中安装jdk1.8

安装tar.gz包：cd到解压后路径
```shell script
python setup.py install
```

下载包
```shell script
wget http://www.bzip.org/1.0.6/bzip2-1.0.6.tar.gz
```

##本文适用于Python 3.8 提示ModuleNotFoundError: No module named '_bz2'错误
```python
ModuleNotFoundError: No module named '_bz2'

该错误是缺失_bz2.cpython-38-x86_64-linux-gnu.so这个os文件，处理步骤如下：

1）下载该文件: https://pan.baidu.com/s/1iPuEBYnUABWf94QM9fQZgQ 提取码: nw2g

2）将下载后的文件放到python3.8文件夹里/usr/local/python/lib/python3.8/lib-dynload/目录下；

在lib-dynload目录下使用"chmod +x _bz2.cpython-38-x86_64-linux-gnu.so"增加该文件的可执行权限

3）再次运行程序可能还会报错：ImportError: libbz2.so.1.0: cannot open shared object file: No such file or directory

1.首先需要使用sudo yum install -y bzip2* 确保系统已经安装了相关的库；

2.此时会发现在/usr/lib64目录下会发现其实有libbz2.so.1.0.6这样一个文件，我们只需要在该目录下使用命令

"sudo ln -s libbz2.so.1.0.6 libbz2.so.1.0"创建一个该文件的软连接。

这样就OK了
```


pytest --version
如果提示没找到pytest，可以在环境变量中配置pytest所在的包
```shell script
vi /etc/profile
在最后追加一句：
export PATH=$PATH:/usr/local/python3/lib/python3.6/site-packages/
：wq保存后退出
source /etc/profile
此时在pytest --version可以查看版本号
```
