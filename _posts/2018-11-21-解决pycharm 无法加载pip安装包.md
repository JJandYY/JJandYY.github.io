---
layout: post
title: "pycharm无法加载python安装包"
data: 2018-11-21 10:52:00 +0800
author: "YY"
tags:
    - Bug之路
---

1. Linux下打开　**/opt/pycharm-xxxxx/helpers/packaging_tool.py**   

  windows下打开　**C:\Program Files\JetBrains\PyCharm 2017.1.1\helpers\packaging_tool.py**
2. 找到下方代码：

```python
def do_install(pkgs):
    try:
        import pip
    except ImportError:
        error_no_pip()
    return pip.main(['install'] + pkgs)

def do_uninstall(pkgs):
   try:
        import pip
    except ImportError:
        error_no_pip()
    return pip.main(['uninstall', '-y'] + pkgs)
```

3. 修改为：

```python
def do_install(pkgs):
    try: 
        from pip._internal import main 
    except Exception: 
        from pip import main 
    except ImportError: 
        error_no_pip()
    return main(['install'] + pkgs)

def do_uninstall(pkgs): 
    try: 
        from pip._internal import main 
    except Exception: 
        from pip import main 
    except ImportError: 
        error_no_pip() 
    return main(['uninstall', '-y'] + pkgs)
```

4. 此时打开pycharm 就能加载到pip安装包了。