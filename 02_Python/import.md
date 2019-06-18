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