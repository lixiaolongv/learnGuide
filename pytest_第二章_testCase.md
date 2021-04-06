# 学习目的：
* 1 如何借助类，模块，目录来组织测试
* 2 使用marker来标记希望同时运行的测试
* 3 使用marker来标记希望跳过运行的测试
* 4 使用marker来标记预期失败的测试
* 5 参数化测试，以便使用多组数据开展测试

## __init__的作用：
* 1 如果放在包目录下，那么它将告诉python解释器，该目录是python包
* 2 如果放在包目录下，同时也为python提供搜索路径

## pytest.ini的作用：
* 1 pytest.ini是可选的
* 2 pytest.ini的作用是保存pytest在该目录下的特定配置，项目中顶多包含一个配置文件，其中的指令可以调节pytest的工作行为。

## conftest.py的作用：
* 1 conftest.py是可选的
* 2 它是pytest的本地插件库
* 3 它可以包含hook函数和fixture函数
* 4 hook函数可以将自定义逻辑引入pytest,用于改善pytest的执行流程
* 5 fixture函数则是用于测试前后执行配置及其销毁逻辑的外壳函数，可以传递测试中用到的资源
* 6 conftest.py同一个目录可以包含多个该文件

## pip install -e
安装包之后，希望修改源码重新安装，就需要-e(editable)选项
例如：pip install -e ./task_proj/

## 使用assert声明
pytest允许在assert关键字后面添加任何表达式(assert experession)
如果表达式的值通过bool转换之后等于False，则意味着测试失败

由于原生pytest assert关键字具有局限性，需要重写assert关键字
运行原理是：pytest会截断对原生assert的调用，替换为pytest定义的assert，从而提供更多的失败信息和细节
查看例子：pytest.org 

## 预期异常
```python
def test_add_raises():
    """add() should raise an exception with wrong type param."""
    with pytest.raises(TypeError):
        tasks.add(task='not a Task object')
```
解析：【只检测了类型异常】如果测试通过，则说明确实发生了我们预期的TypeError异常，如果抛出其他类型的异常，则说明与我们预期不一致，测试失败

```python
def test_start_tasks_db_raises():
    """Make sure unsupported db raises an exception."""
    with pytest.raises(ValueError) as excinfo:
        tasks.start_tasks_db('some/great/path', 'mysql')
    exception_msg = excinfo.value.args[0]
    assert exception_msg == "db_type must be a 'tiny' or 'mongo'"
```
解析：为检测异常消息是否符合预期，可以通过增加as excinfo语句来得到异常信息的值，在进行比对


## 测试函数的标记
* 1 允许使用marker对测试函数进行标记
* 2 一个测试函数可以有多个marker
* 3 一个marker也可以标记多个测试函数
```python
@pytest.mark.smoke
def test_list_raises():
    """list() should raise an exception with wrong type param."""
    with pytest.raises(TypeError):
        tasks.list_tasks(owner=123)

@pytest.mark.get
@pytest.mark.smoke
def test_get_raises():
    """get() should raise an exception with wrong type param."""
    with pytest.raises(TypeError):
        tasks.get(task_id='123')
```
解析：使用命令中指定 -m marker_name，就可以运行他们，-m后面也可以使用表达式，可以在标记之间添加and,or,not关键字
``pytest -v -m 'smoke and get' test_api.py``  
注意：  
[pytest]  
markers =  
        smoke   
        get 

## 跳过测试
* 1 skip和skipif 允许你跳过不希望运行的测试，case不会被执行
```python
@pytest.mark.skipif(tasks.__version__ < '0.2.0',
                    reason='not supported until version 0.2.0')
def test_unique_id_1():
    """Calling unique_id() twice should return different numbers."""
    id_1 = tasks.unique_id()
    id_2 = tasks.unique_id()
    assert id_1 != id_2
```
解析：使用pytest.mark.skipif(experssion, reson) 就可以跳过测试,使用命令``pytest -rs test_api.py``
可以查看跳过原因，具体使用参数可查看``pytest --help``

## 标记预期会失败的测试
* 1 使用xfail标记，则告诉pytest运行此测试，预期该case会失败
```python
@pytest.mark.xfail(tasks.__version__ < '0.2.0',
                   reason='not supported until version 0.2.0')
def test_unique_id_1():
    """Calling unique_id() twice should return different numbers."""
    id_1 = tasks.unique_id()
    id_2 = tasks.unique_id()
    assert id_1 != id_2
```
解析：使用xfail，x代表XFAIL,意味着"expected to fail"(预期失败，实际上也失败了)
大写的X代表XPASS，意味着"expected to fail but passed" (预期失败但是实际运行并没有失败)  
``特殊点：对于标记为XFAIL，但是实际运行结果是XPASS的测试，可以在pytest配置中强制指定结果为FAIL，修改pytest.ini即可``  
[pytest]  
xfail_strict=true

## 运行测试子集
* 1 单个目录
* 2 单个测试文件/模快
* 3 单个测试函数
* 4 单个测试类
* 5 单个测试类中的测试方法
* 6 用测试名划分测试集合

运行单个目录：``pytest -v tests/func --tb=no``  
运行单个测试文件/模快: ``pytest -v tests/func/test_model.py``  
运行单个测试函数：``pytest -v tests/func/test_model.py::test_function_01``    
运行单个测试类：``pytest -v tests/func/test_model.py::Testclass``  
运行单个测试类中的测试方法： ``pytest -v tests/func/test_model.py::Testclass::test_function_01``  
运行用测试名划分测试集合：运行会员模块``pytest -v -k _member``  
剔除模块中的部分测试case：``pytest -v -k "_member and not _user"``  
解析：-k选项允许用一个表达式指定需要运行的测试，该表达式可以匹配测试名（或其子串），表达式可以包含and,or,not  

## 参数化测试
```python
@pytest.mark.parametrize('task', [
    pytest.param(Task('create'), id='just summary'),
    pytest.param(Task('inspire', 'Michelle'), id='summary/owner'),
    pytest.param(Task('encourage', 'Michelle', True), id='summary/owner/done')])
def test_add_6(task):
    """Demonstrate pytest.param and id."""
    task_id = tasks.add(task)
    t_from_db = tasks.get(task_id)
    assert equivalent(t_from_db, task)
```

```python
@pytest.mark.parametrize('task', tasks_to_try, ids=task_ids)
class TestAdd():
    """Demonstrate parametrize and test classes."""

    def test_equivalent(self, task):
        """Similar test, just within a class."""
        task_id = tasks.add(task)
        t_from_db = tasks.get(task_id)
        assert equivalent(t_from_db, task)

    def test_valid_id(self, task):
        """We can use the same data for multiple tests."""
        task_id = tasks.add(task)
        t_from_db = tasks.get(task_id)
        assert t_from_db.id == task_id
```






