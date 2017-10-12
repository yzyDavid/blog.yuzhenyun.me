---
title: PyTorch 中 DataLoader 类的实现
date: 2017-10-12 08:31:03
tags:
- PyTorch
- 编程
- 深度学习
---

本文写作时 `PyTorch` 版本为 0.2.0

官方文档：http://pytorch.org/docs/master/data.html

`Dataset` 表达数据集，`Sampler` 表达取样数据的方式，`DataLoader` 将两者组合起来，达到一个可以源源不断提取数据的工厂的效果。所以 `DataLoader` 类只有一个，而 `Sampler` 有很多种。

<!-- more -->

源代码如下，这个类本身还是很简洁的。

```python
class DataLoader(object):
    """
    Data loader. Combines a dataset and a sampler, and provides
    single- or multi-process iterators over the dataset.

    Arguments:
        dataset (Dataset): dataset from which to load the data.
        batch_size (int, optional): how many samples per batch to load
            (default: 1).
        shuffle (bool, optional): set to ``True`` to have the data reshuffled
            at every epoch (default: False).
        sampler (Sampler, optional): defines the strategy to draw samples from
            the dataset. If specified, ``shuffle`` must be False.
        batch_sampler (Sampler, optional): like sampler, but returns a batch of
            indices at a time. Mutually exclusive with batch_size, shuffle,
            sampler, and drop_last.
        num_workers (int, optional): how many subprocesses to use for data
            loading. 0 means that the data will be loaded in the main process
            (default: 0)
        collate_fn (callable, optional): merges a list of samples to form a mini-batch.
        pin_memory (bool, optional): If ``True``, the data loader will copy tensors
            into CUDA pinned memory before returning them.
        drop_last (bool, optional): set to ``True`` to drop the last incomplete batch,
            if the dataset size is not divisible by the batch size. If False and
            the size of dataset is not divisible by the batch size, then the last batch
            will be smaller. (default: False)
    """

    def __init__(self, dataset, batch_size=1, shuffle=False, sampler=None, batch_sampler=None,
                 num_workers=0, collate_fn=default_collate, pin_memory=False, drop_last=False):
        self.dataset = dataset
        self.batch_size = batch_size
        self.num_workers = num_workers
        self.collate_fn = collate_fn
        self.pin_memory = pin_memory
        self.drop_last = drop_last

        if batch_sampler is not None:
            if batch_size > 1 or shuffle or sampler is not None or drop_last:
                raise ValueError('batch_sampler is mutually exclusive with '
                                 'batch_size, shuffle, sampler, and drop_last')

        if sampler is not None and shuffle:
            raise ValueError('sampler is mutually exclusive with shuffle')

        if batch_sampler is None:
            if sampler is None:
                if shuffle:
                    sampler = RandomSampler(dataset)
                else:
                    sampler = SequentialSampler(dataset)
            batch_sampler = BatchSampler(sampler, batch_size, drop_last)

        self.sampler = sampler
        self.batch_sampler = batch_sampler

    def __iter__(self):
        return DataLoaderIter(self)

    def __len__(self):
        return len(self.batch_sampler)
```

这就牵扯到和他相关的 `Dataset` 类以及 `Sampler` 了。

前两个会抛异常的 if 块是为了检测传入的参数搭配是否满足文档中给出的假设。后面的大一点的 if 块是实际的核心逻辑。

下面给出 `Sampler` 基类的源代码，这是一个抽象类：

```python
class Sampler(object):
    """Base class for all Samplers.

    Every Sampler subclass has to provide an __iter__ method, providing a way
    to iterate over indices of dataset elements, and a __len__ method that
    returns the length of the returned iterators.
    """

    def __init__(self, data_source):
        pass

    def __iter__(self):
        raise NotImplementedError

    def __len__(self):
        raise NotImplementedError
```

所有的 Sampler 都需要重写也只需要重写这三个魔法方法。这是官方提供的所有 Sampler 所共同的特点。要注意的是，构造函数接受的参数可以不同。这一点给了取样器足够的灵活性。从 `DataLoader` 的实现中可以看出，不指定 Sampler 而指定 Shuffle 等等几个参数，实际实现就是使用了官方的 RandomSampler 等等方式。

待证实：这里的 sampler 参数接收的应该是对象而非是类。所以自己提供取样器实现的话，可以自己控制构造函数参数，将构造好的对象传入即可。

要注意 `BatchSampler` 是对其他 sampler 的一个封装。`DataLoader` 会同时生成一个 Sampler 以及一个使用它封装的 BatchSampler。当然前提条件是你没有显式指定 BatchSampler。