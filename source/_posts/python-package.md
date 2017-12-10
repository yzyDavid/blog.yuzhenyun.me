---
title: Python 包与路径相关的一堆坑
date: 2017-09-21 15:37:42
tags:
- Python
- 编程
---

Python 模块相关的细节：

- Python 2/3 区别
- 绝对路径导入/相对路径导入
- 模块启动/单文件启动
- 包/模块
- `__name__ == '__main__'`

参考文档：http://www.pythondoc.com/pythontutorial3/modules.html

<!-- more -->

---------------------------------

```shell
python -m foo/bar
python foo/bar.py
```

以上两种方式会导致`sys.path`值不一致。作为模块启动时当前目录会被加进 path，否则文件目录加进入 path。造成的影响是：

模块启动 - 可以相对路径导入自己目录的其他模块

文件启动 - 不可以

-----------------------------------

Python 2 - 默认相对路径导入

Python 3 - 默认绝对路径导入

-----------------------------------

包 - 含有`__init__.py`的目录

模块 - `.py`文件

-------------------------

同一级之中的导入：

```python
from foo import bar  # PyCharm Python3 报错，命令行单文件启动可以，没弄明白
from .foo import bar  # PyCharm 不报错，需要模块启动
```

-------------------------

有什么坑踩到了再来总结。