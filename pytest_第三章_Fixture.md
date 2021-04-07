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

## 使用--setup-show回嗍fixture的执行过程
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

## 使用fixture传递测试数据
* 1 fixture非常适合存放测试数据，并且它可以返回任何数据。

```python
@pytest.fixture()
def a_tuple():
    """Return something more interesting."""
    return (1, 'foo', None, {'bar': 23})


def test_a_tuple(a_tuple):
    """Demo the a_tuple fixture."""
    assert a_tuple[3]['bar'] == 32
```
解析：fixture可以返回数组，并且不需要把fixture与test_测试函数放在不同文件，相当灵活，可自由搭配组合测试。

```python
@pytest.fixture()
def some_other_data():
    """Raise an exception from fixture."""
    x = 43
    assert x == 42
    return x


def test_other_data(some_other_data):
    """Try to use failing fixture."""
    assert some_data == 42
```
## 返回数据如下：
```python
collected 1 item                                                                                                                                                                                    

test_fixtures.py::test_other_data ERROR                                                                                                                                                       [100%]

============================================================================================== ERRORS ===============================================================================================
_________________________________________________________________________________ ERROR at setup of test_other_data _________________________________________________________________________________

    @pytest.fixture()
    def some_other_data():
        """Raise an exception from fixture."""
        x = 43
>       assert x == 42
E       assert 43 == 42
E         +43
E         -42

test_fixtures.py:21: AssertionError
====================================================================================== short test summary info ======================================================================================
ERROR test_fixtures.py::test_other_data - assert 43 == 42
========================================================================================= 1 error in 0.08s =================================================================
```
解析：
* 1 堆栈正确定位了fixture函数中assert发生的异常
* 2 test_other_data并没有被报告为Fail，而是Error，说明错误不是发生在核心测试函数内，而是其依赖的fixture内
* 3 只有Fail，才能说明是核心测试函数测试失败

## 使用多个fixture
* 1 fixture之间可以嵌套使用
```python
@pytest.fixture()
def tasks_db(tasks_db_session):
    """An empty tasks db."""
    tasks.delete_all()


@pytest.fixture()
def db_with_3_tasks(tasks_db, tasks_just_a_few):
    """Connected db with 3 tasks, all unique."""
    for t in tasks_just_a_few:
        tasks.add(t)


@pytest.fixture()
def db_with_multi_per_owner(tasks_db, tasks_mult_per_owner):
    """Connected db with 9 tasks, 3 owners, all with 3 tasks."""
    for t in tasks_mult_per_owner:
        tasks.add(t)
```

## 指定fixture的作用范围
```python
@pytest.fixture(scope='function')
def func_scope():
    """A function scope fixture."""


@pytest.fixture(scope='module')
def mod_scope():
    """A module scope fixture."""


@pytest.fixture(scope='session')
def sess_scope():
    """A session scope fixture."""


@pytest.fixture(scope='class')
def class_scope():
    """A class scope fixture."""


def test_1(sess_scope, mod_scope, func_scope):
    """Test using session, module, and function scope fixtures."""


def test_2(sess_scope, mod_scope, func_scope):
    """Demo is more fun with multiple tests."""

@pytest.mark.usefixtures('class_scope')
class TestSomething():
    """Demo class scope fixtures."""

    def test_3(self):
        """Test using a class scope fixture."""

    def test_4(self):
        """Again, multiple tests are more fun."""
```
解析：
* 1 @pytest.fixture包含scope的可选参数，用于控制fixture执行配置和销毁逻辑的频率
* 2 scope的选项有function，module，session，class，默认值为function，未指定也默认为函数级别
* 3 scope="function"，函数级别，每个测试函数只需要运行一次
* 4 scope="module"，模块级别，每个模块只需要运行一次，无论模块里有多少个测试函数，类方法或者其他的fixture都可以共享这个fixture
* 5 scope="class"，类级别，每个测试类只需要运行一次，无论测试类里有多少类方法都可以共享这个fixture
* 6 scope="session"，会话级别，每次会话只需要运行一次，一次会话中的所有测试函数，方法都可以共享这个fixture
* 7 注意：usefixtures

## 测试结果如下
```python
collected 4 items                                                                                                                                                                                   

test_scope.py 
SETUP    S sess_scope
    SETUP    M mod_scope
        SETUP    F func_scope
        test_scope.py::test_1 (fixtures used: func_scope, mod_scope, sess_scope).
        TEARDOWN F func_scope
        SETUP    F func_scope
        test_scope.py::test_2 (fixtures used: func_scope, mod_scope, sess_scope).
        TEARDOWN F func_scope
      SETUP    C class_scope
        test_scope.py::TestSomething::test_3 (fixtures used: class_scope).
        test_scope.py::TestSomething::test_4 (fixtures used: class_scope).
      TEARDOWN C class_scope
    TEARDOWN M mod_scope
TEARDOWN S sess_scope

========================================================================================= 4 passed in 0.05s ====================================
```
解析：
* 1 fixture只能使用同级别的fixture，或者比自己级别更高的fixture！
* 2 函数级别的fixture可以使用同级别的fixture，也可以使用类级别，模块级别，会话级别的fixture，但是不能返过来！
* 3 对于经常需要使用的全局变量，可以设置fixture为会话级别，每次会话只需要调用一次即可！

## 修改fixture的作用范围
```python
@pytest.fixture(scope='session')
def tasks_db_session(tmpdir_factory):
    """Connect to db before tests, disconnect after."""
    temp_dir = tmpdir_factory.mktemp('temp')
    tasks.start_tasks_db(str(temp_dir), 'tiny')
    yield
    tasks.stop_tasks_db()


@pytest.fixture()
def tasks_db(tasks_db_session):
    """An empty tasks db."""
    tasks.delete_all()
```
解析：  
每次测试都需要配置一个临时目录，新建一次数据库连接，这样显得有些浪费，实际上，只要保证每次测试时数据库连接一次即可，
所以把数据库的连接调整至会话级别即可，所以选择会话级别的临时工厂tmpdir_factory

## 使用usefixtures指定fixture
```python
@pytest.mark.usefixtures('class_scope')
class TestSomething():
    """Demo class scope fixtures."""

    def test_3(self):
        """Test using a class scope fixture."""

    def test_4(self):
        """Again, multiple tests are more fun."""
```
解析：  
使用@pytest.mark.usefixtures('fixture1', 'fixrure2')标记测试函数或类，与在测试函数中添加fixture的区别是
只有后者才能使用fixture的返回值

## 为常用的fixture添加autouse选项
```python
@pytest.fixture(autouse=True, scope='session')
def footer_session_scope():
    """Report the time at the end of a session."""
    yield
    now = time.time()
    print('--')
    print('finished : {}'.format(time.strftime('%d %b %X', time.localtime(now))))
    print('-----------------')


@pytest.fixture(autouse=True)
def footer_function_scope():
    """Report test durations after each function."""
    start = time.time()
    yield
    stop = time.time()
    delta = stop - start
    print('\ntest duration : {:0.3} seconds'.format(delta))


def test_1():
    """Simulate long-ish running test."""
    time.sleep(1)


def test_2():
    """Simulate slightly longer test."""
    time.sleep(1.23)
```

测试结果如下：
```python
collected 2 items                                                                                                                                                                                   

test_autouse.py::test_1 PASSED
test duration : 1.0 seconds

test_autouse.py::test_2 PASSED
test duration : 1.23 seconds
--
finished : 07 Apr 22:39:52
-----------------
```
解析：  
* 1 指定autouse=true选项，使作用域内的测试函数都运行该fixture

## 为fixture重命名
```python
@pytest.fixture(name='lue')
def ultimate_answer_to_life_the_universe_and_everything():
    """Return ultimate answer."""
    return 42


def test_everything(lue):
    """Use the shorter name."""
    assert lue == 42
```

测试结果如下：
```python
(homedotest)  mac@192  ~/Downloads/code/ch3  pytest --setup-show test_rename_fixture.py 
======================================================================================== test session starts ========================================================================================
platform darwin -- Python 3.8.2, pytest-6.2.2, py-1.10.0, pluggy-0.13.1
rootdir: /Users/mac/Downloads/code/ch3
plugins: rerunfailures-9.1.1, html-3.1.1, xdist-2.2.1, metadata-1.11.0, ordering-0.6, allure-pytest-2.8.36, forked-1.3.0
collected 1 item                                                                                                                                                                                    

test_rename_fixture.py 
        SETUP    F lue
        test_rename_fixture.py::test_everything (fixtures used: lue).
        TEARDOWN F lue

========================================================================================= 1 passed in 0.02s ==============
```
解析：重命名之后，fixture就是lue

查看所有fixture的列表
```python
(homedotest)  ✘ mac@192  ~/Downloads/code/ch3/b/tasks_proj  pytest --fixtures tests/func/test_add.py 
======================================================================================== test session starts ========================================================================================
platform darwin -- Python 3.8.2, pytest-6.2.2, py-1.10.0, pluggy-0.13.1
rootdir: /Users/mac/Downloads/code/ch3/b/tasks_proj/tests, configfile: pytest.ini
plugins: rerunfailures-9.1.1, html-3.1.1, xdist-2.2.1, metadata-1.11.0, ordering-0.6, allure-pytest-2.8.36, forked-1.3.0
collected 3 items                                                                                                                                                                                   
cache
    Return a cache object that can persist state between testing sessions.
    
    cache.get(key, default)
    cache.set(key, value)
    
    Keys must be ``/`` separated strings, where the first part is usually the
    name of your plugin or application to avoid clashes with other cache users.
    
    Values can be any object handled by the json stdlib module.

capsys
    Enable text capturing of writes to ``sys.stdout`` and ``sys.stderr``.
    
    The captured output is made available via ``capsys.readouterr()`` method
    calls, which return a ``(out, err)`` namedtuple.
    ``out`` and ``err`` will be ``text`` objects.

capsysbinary
    Enable bytes capturing of writes to ``sys.stdout`` and ``sys.stderr``.
    
    The captured output is made available via ``capsysbinary.readouterr()``
    method calls, which return a ``(out, err)`` namedtuple.
    ``out`` and ``err`` will be ``bytes`` objects.

capfd
    Enable text capturing of writes to file descriptors ``1`` and ``2``.
    
    The captured output is made available via ``capfd.readouterr()`` method
    calls, which return a ``(out, err)`` namedtuple.
    ``out`` and ``err`` will be ``text`` objects.

capfdbinary
    Enable bytes capturing of writes to file descriptors ``1`` and ``2``.
    
    The captured output is made available via ``capfd.readouterr()`` method
    calls, which return a ``(out, err)`` namedtuple.
    ``out`` and ``err`` will be ``byte`` objects.

doctest_namespace [session scope]
    Fixture that returns a :py:class:`dict` that will be injected into the
    namespace of doctests.

pytestconfig [session scope]
    Session-scoped fixture that returns the :class:`_pytest.config.Config` object.
    
    Example::
    
        def test_foo(pytestconfig):
            if pytestconfig.getoption("verbose") > 0:
                ...

record_property
    Add extra properties to the calling test.
    
    User properties become part of the test report and are available to the
    configured reporters, like JUnit XML.
    
    The fixture is callable with ``name, value``. The value is automatically
    XML-encoded.
    
    Example::
    
        def test_function(record_property):
            record_property("example_key", 1)

record_xml_attribute
    Add extra xml attributes to the tag for the calling test.
    
    The fixture is callable with ``name, value``. The value is
    automatically XML-encoded.

record_testsuite_property [session scope]
    Record a new ``<property>`` tag as child of the root ``<testsuite>``.
    
    This is suitable to writing global information regarding the entire test
    suite, and is compatible with ``xunit2`` JUnit family.
    
    This is a ``session``-scoped fixture which is called with ``(name, value)``. Example:
    
    .. code-block:: python
    
        def test_foo(record_testsuite_property):
            record_testsuite_property("ARCH", "PPC")
            record_testsuite_property("STORAGE_TYPE", "CEPH")
    
    ``name`` must be a string, ``value`` will be converted to a string and properly xml-escaped.
    
    .. warning::
    
        Currently this fixture **does not work** with the
        `pytest-xdist <https://github.com/pytest-dev/pytest-xdist>`__ plugin. See issue
        `#7767 <https://github.com/pytest-dev/pytest/issues/7767>`__ for details.

caplog
    Access and control log capturing.
    
    Captured logs are available through the following properties/methods::
    
    * caplog.messages        -> list of format-interpolated log messages
    * caplog.text            -> string containing formatted log output
    * caplog.records         -> list of logging.LogRecord instances
    * caplog.record_tuples   -> list of (logger_name, level, message) tuples
    * caplog.clear()         -> clear captured records and formatted log output string

monkeypatch
    A convenient fixture for monkey-patching.
    
    The fixture provides these methods to modify objects, dictionaries or
    os.environ::
    
        monkeypatch.setattr(obj, name, value, raising=True)
        monkeypatch.delattr(obj, name, raising=True)
        monkeypatch.setitem(mapping, name, value)
        monkeypatch.delitem(obj, name, raising=True)
        monkeypatch.setenv(name, value, prepend=False)
        monkeypatch.delenv(name, raising=True)
        monkeypatch.syspath_prepend(path)
        monkeypatch.chdir(path)
    
    All modifications will be undone after the requesting test function or
    fixture has finished. The ``raising`` parameter determines if a KeyError
    or AttributeError will be raised if the set/deletion operation has no target.

recwarn
    Return a :class:`WarningsRecorder` instance that records all warnings emitted by test functions.
    
    See http://docs.python.org/library/warnings.html for information
    on warning categories.

tmpdir_factory [session scope]
    Return a :class:`_pytest.tmpdir.TempdirFactory` instance for the test session.

tmp_path_factory [session scope]
    Return a :class:`_pytest.tmpdir.TempPathFactory` instance for the test session.

tmpdir
    Return a temporary directory path object which is unique to each test
    function invocation, created as a sub directory of the base temporary
    directory.
    
    By default, a new base temporary directory is created each test session,
    and old bases are removed after 3 sessions, to aid in debugging. If
    ``--basetemp`` is used then it is cleared each session. See :ref:`base
    temporary directory`.
    
    The returned object is a `py.path.local`_ path object.
    
    .. _`py.path.local`: https://py.readthedocs.io/en/latest/path.html

tmp_path
    Return a temporary directory path object which is unique to each test
    function invocation, created as a sub directory of the base temporary
    directory.
    
    By default, a new base temporary directory is created each test session,
    and old bases are removed after 3 sessions, to aid in debugging. If
    ``--basetemp`` is used then it is cleared each session. See :ref:`base
    temporary directory`.
    
    The returned object is a :class:`pathlib.Path` object.


---------------------------------------------------------------------------------- fixtures defined from conftest -----------------------------------------------------------------------------------
tasks_db
    An empty tasks db.

tasks_just_a_few [session scope]
    All summaries and owners are unique.

tasks_mult_per_owner [session scope]
    Several owners with several tasks each.

db_with_3_tasks
    Connected db with 3 tasks, all unique.

db_with_multi_per_owner
    Connected db with 9 tasks, 3 owners, all with 3 tasks.

tasks_db_session [session scope]
    Connect to db before tests, disconnect after.


----------------------------------------------------------------------------- fixtures defined from pytest_html.plugin ------------------------------------------------------------------------------
extra
    Add details to the HTML reports.
    
    .. code-block:: python
    
        import pytest_html
    
    
        def test_foo(extra):
            extra.append(pytest_html.extras.url("http://www.example.com/"))


--------------------------------------------------------------------------- fixtures defined from pytest_metadata.plugin ----------------------------------------------------------------------------
metadata [session scope]
    Provide test session metadata

include_metadata_in_junit_xml [session scope]
    Provide test session metadata


-------------------------------------------------------------------------------- fixtures defined from xdist.plugin ---------------------------------------------------------------------------------
worker_id [session scope]
    Return the id of the current worker ('gw0', 'gw1', etc) or 'master'
    if running on the master node.

testrun_uid [session scope]
    Return the unique id of the current test.


======================================================================================= no tests ran in 0.08s =====
```

## fixture的参数化
```python
tasks_to_try = (Task('sleep', done=True),
                Task('wake', 'brian'),
                Task('breathe', 'BRIAN', True),
                Task('exercise', 'BrIaN', False))

task_ids = ['Task({},{},{})'.format(t.summary, t.owner, t.done)
            for t in tasks_to_try]


def equivalent(t1, t2):
    """Check two tasks for equivalence."""
    return ((t1.summary == t2.summary) and
            (t1.owner == t2.owner) and
            (t1.done == t2.done))


@pytest.fixture(params=tasks_to_try)
def a_task(request):
    """Using no ids."""
    return request.param


def test_add_a(tasks_db, a_task):
    """Using a_task fixture (no ids)."""
    task_id = tasks.add(a_task)
    t_from_db = tasks.get(task_id)
    assert equivalent(t_from_db, a_task)
```

## 参数化fixture



