---
title: tf.pad 通俗解释
date: 2017-09-20 15:49:48
tags:
- tensorflow
- 深度学习
---

```python
pad(
    tensor,
    paddings,
    mode='CONSTANT',
    name=None,
    constant_values=0
)
```

关键的就是其中`paddings`这个参数，接受一个二维张量，且需要为 [n, 2] 这种 shape。

举个栗子：

```python
[
    [
        1, # 第0维，前面补1个
      	2  # 第0维，后面补2个
    ],
    [
     	3, # 第1维，前面补3个
      	4  # 第2维，后面补4个
    ]
]
```

有几个维度，就要几个数对，每个数对指示那个维度上前后分别补几个。

<!-- more -->

中文文档不能看，毕竟更新不及时啊，还是来看原文吧 [https://www.tensorflow.org/api_docs/python/tf/pad](https://www.tensorflow.org/api_docs/python/tf/pad)

`mode`参数现在有了 3 种取值，分别为`CONSTANT` `REFLECT` `SYMMETRIC`，其中REFLECT就是回环补齐。

还是放一下官方文档那个例子吧，挺清楚的。

```python
# 't' is [[1, 2, 3], [4, 5, 6]].
# 'paddings' is [[1, 1,], [2, 2]].
# 'constant_values' is 0.
# rank of 't' is 2.
pad(t, paddings, "CONSTANT") ==> [[0, 0, 0, 0, 0, 0, 0],
                                  [0, 0, 1, 2, 3, 0, 0],
                                  [0, 0, 4, 5, 6, 0, 0],
                                  [0, 0, 0, 0, 0, 0, 0]]

pad(t, paddings, "REFLECT") ==> [[6, 5, 4, 5, 6, 5, 4],
                                 [3, 2, 1, 2, 3, 2, 1],
                                 [6, 5, 4, 5, 6, 5, 4],
                                 [3, 2, 1, 2, 3, 2, 1]]

pad(t, paddings, "SYMMETRIC") ==> [[2, 1, 1, 2, 3, 3, 2],
                                   [2, 1, 1, 2, 3, 3, 2],
                                   [5, 4, 4, 5, 6, 6, 5],
                                   [5, 4, 4, 5, 6, 6, 5]]
```

另外`Keras`的后端封装当中没有对应的函数，是个小遗憾。