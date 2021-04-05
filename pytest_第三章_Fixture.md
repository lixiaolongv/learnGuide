# 学习目的：
* 1 fixture是在函数运行前后，由pytest执行的外壳函数
* 2 fixture可以定制，满足多变的测试需求
* 3 定义传入测试中的数据集
* 4 配置测试前系统的初始状态
* 5 为批量数据提供数据源

## 通过conftest.py共享fixture
* 1 conftest.py文件可以有多个，其作用域可以划分是：
    * a 根目录：作用域是全局，所有的测试文件均可共享
    * b 子目录：作用域是该目录及其子目录下测试共享
* 2 conftest.py不能被测试文件导入
* 3 conftest.py被pytest视为本地插件库

## 使用fixture执行配置及其销毁逻辑
```python
@pytest.fixture()
def tasks_db(tmpdir):
    """Connect to db before tests, disconnect after."""
    # Setup : start db
    tasks.start_tasks_db(str(tmpdir), 'tiny')

    yield  # this is where the testing happens

    # Teardown : stop db
    tasks.stop_tasks_db()
```
解析：
* 1 语法规则：函数上标记@pytest.fixture()，表示该函数为fixture
* 2 tmpdir参数：临时目录，是pytest内置的一个fixture
* 3 执行流程：fixture会在测试函数执行之前运行，但是如果fixture函数包含yield，那么系统会在yield处停止，转而运行测试函数，
等待测试函数执行完毕之后再回到fixture,继续执行yield之后的代码
* 4 setup：将yield之前的代码视为配置（setup）过程
* 5 teardown：将yield之后的代码视为（teardown）过程
* 6 无论测试过程中发生了什么，yield之后的代码都会被执行
* 7 yield可以直接返回数据，之后还可以继续执行测试流程，与return不一样，返回数据之后，将不再执行

```python
def test_add_returns_valid_id(tasks_db):
    """tasks.add(<valid task>) should return an integer."""
    # GIVEN an initialized tasks db
    # WHEN a new task is added
    # THEN returned task_id is of type int
    new_task = Task('do something')
    task_id = tasks.add(new_task)
    assert isinstance(task_id, int)
```
解析：   
使用GIVEN-WHEN-THEN这样的注释来组织测试结构，可以更清晰的理清楚测试逻辑

## 使用--setup-shouw回嗍fixture的执行过程
```python
(homedotest)  mac@192  ~/Downloads/code/ch3/a/tasks_proj/tests/func  pytest --setup-show test_add.py -k valid_id            
======================================================================================== test session starts ========================================================================================
platform darwin -- Python 3.8.2, pytest-6.2.2, py-1.10.0, pluggy-0.13.1
rootdir: /Users/mac/Downloads/code/ch3/a/tasks_proj/tests, configfile: pytest.ini
plugins: rerunfailures-9.1.1, html-3.1.1, xdist-2.2.1, metadata-1.11.0, ordering-0.6, allure-pytest-2.8.36, forked-1.3.0
collected 3 items / 2 deselected / 1 selected                                                                                                                                                       

test_add.py 
SETUP    S tmp_path_factory
        SETUP    F tmp_path (fixtures used: tmp_path_factory)
        SETUP    F tmpdir (fixtures used: tmp_path)
        SETUP    F tasks_db (fixtures used: tmpdir)
        func/test_add.py::test_add_returns_valid_id (fixtures used: request, tasks_db, tmp_path, tmp_path_factory, tmpdir).
        TEARDOWN F tasks_db
        TEARDOWN F tmpdir
        TEARDOWN F tmp_path
TEARDOWN S tmp_path_factory

================================================================================== 1 passed, 2 deselected in 0.04s ====================================================================
```
解析：
* 1 测试函数夹在最中间
* 2 pytest将每一个fixture的执行分成SETUP和TEARDOWN两部分
* 3 注意：tmpdir-tmp_path-tmp_path_factory的关系
* 4 fixture名称前的F和S代表的是fixture的作用范围，F代表函数级别的作用范围，S代表会话级别的作用范围


