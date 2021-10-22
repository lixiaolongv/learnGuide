# Poetry

poetry是一个Python虚拟环境和依赖管理的工具，另外还提供了打包和发布的功能！
该工具是未来Python开发项目的方向！

官方文档：https://python-poetry.org/

windows10的操作系统：
进入打开win10的powershell，输入命令
powershell

#名称 功能
poetry new #创建一个项目脚手架，包含基本结构、pyproject.toml 文件  
poetry init #基于已有的项目代码创建 pyproject.toml 文件，支持交互式填写   
poetry install #安装依赖库   
poetry update #更新依赖库   
poetry add #添加依赖库   
poetry remove #移除依赖库   
poetry show #查看具体依赖库信息，支持显示树形依赖链   
poetry build #构建 tar.gz 或 wheel 包   
poetry publish #发布到 PyPI   
poetry run #运行脚本和代码  
————————————————
原文链接：https://blog.csdn.net/zzq900503/article/details/93083065

安装poetry：   
1，输入安装命令：   
(Invoke-WebRequest -proxy http://10.5.17.45:8118 -Uri https://raw.githubusercontent.com/python-poetry/poetry/master/get-poetry.py -UseBasicParsing).Content | python

2，查看是否安装成功   
poetry --version

poetry官网：  
https://python-poetry.org/

3，pyproject.toml到底是什么东西?   
https://python.freelycode.com/contribution/detail/1910

4，poetry与pycharm集成   
https://github.com/koxudaxi/poetry-pycharm-plugin

5，为python项目启用peotry和pyproject.toml【博客很漂亮，值得借鉴】
https://www.cnblogs.com/dongfangtianyu/p/14382420.html



mac安装命令：
curl -sSL https://raw.githubusercontent.com/sdispater/poetry/master/get-poetry.py | python