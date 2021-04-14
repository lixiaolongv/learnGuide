# 学习目的
* 1 介绍内置的fixture以及使用方法

## 使用tempdir和tempdir_factory
* 1 tempdir的作用范围是fuction级别
* 2 tempdir_factory的作用范围是session级别
* 3 负责在测试开始运行前创建临时文件目录，并在测试结束之后删除
* 4 tempdir的返回值是 py.path.local 类型的一个对象

## 使用场景
* 1 测试代码需要对文件进行读写操作
* 2 单个测试使用tempdir，多个测试使用tempdir_factory
* 3 针对A接口返回的response作为B接口请求的入参

## 数据存储
* 1 使用collections中的namedtuple来存储，把测试中的response值当成全局变量来存储
* 2 使用临时数据库TinyDB来存储数据
