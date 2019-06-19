# Python import

## SystemError: Parent module '' not loaded, cannot perform relative import

今天在调试时碰到了这个问题，根本原因是我把Python包中的某个Py文件当作脚本直接执行了，在这个Py文件中使用relative import就会报上述的错误。running a module inside the package as a script

比如：
```
main.py
mypackage/
    __init__.py
    mymodule.py
    myothermodule.py
```


在`myothermodule.py`里面

```
#!/usr/bin/env python3

from .mymodule import as_int

# Exported function
def add(a, b):
    return as_int(a) + as_int(b)

# Test function for module  
def _test():
    assert add('1', '1') == 2

if __name__ == '__main__':
    _test()
```

运行这个py就会报错。

引用龟叔的一段话，单独执行module中的某个script是反设计模式的，通常不要这么干，在module外来调用module中的script定义的变量等内容。

I'm -1 on this and on any other proposed twiddlings of the __main__ machinery. The only use case seems to be running scripts that happen to be living inside a module's directory, which I've always seen as an antipattern. To make me change my mind you'd have to convince me that it isn't.

https://stackoverflow.com/questions/16981921/relative-imports-in-python-3


## Python模块内部正确的import
当代码组织成为python package后，如果在这个package内一个Module想要导入另一个也在这个pakcage中的Module，可以考虑relative import

```
mypackage/
    __init__.py
    A/
        __init__.py
        spam.py
        grok.py
    B/
        __init__.py
        bar.py
```

### 如果模块mypackage.A.spam要导入同目录下的模块grok

```
# mypackage/A/spam.py
from . import grok
```

### 如果模块mypackage.A.spam要导入不同目录下的模块B.bar

```
# mypackage/A/spam.py
from ..B import bar
```

### 语法
* 对于python3, 在包内，既可以使用相对路径也可以使用绝对路径来导入
```
# mypackage/A/spam.py
from mypackage.A import grok # OK
from . import grok # OK
import grok # Error (not found)
```
* .为当前目录，..B为目录../B
* 这种语法只适用于import
```
from . import grok # OK
import .grok # ERROR
```
* 使用相对导入看起来像是浏览文件系统，但是不能到定义包的目录之外
* relative import 只适用于library中Module之间的导入，如果要单独运行library中某个module中的某个python文件，该文件中的relative import就会报错，这个错误就是本文最开头的错误
```
% python3 mypackage/A/spam.py # Relative imports fail
```
解决办法是有的，使用`python -m`选项，学习了：
```
% python3 -m mypackage.A.spam # Relative imports work
```