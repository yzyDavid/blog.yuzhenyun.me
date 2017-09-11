---
title: Scrapy 爬虫框架源码架构初探
date: 2017-09-09 16:53:04
tags:
- 编程
- Python
---

## 简介

Scrapy 作为最负盛名的 Python 爬虫框架，本身以及强大的中间件机制提供了爬虫管理的方方面面，让开发者只需专注于网页解析的逻辑本身。本文从离用户最近的代码出发稍稍探索一下这个框架。

<!-- more -->

- 环境：

  - Python 3.6.1
  - Scrapy 1.4.0

  如无特殊说明，本文中的源代码均摘录于此环境，为说明方便，可能会删减改动。

官方文档：https://docs.scrapy.org/en/latest/index.html    此链接总是指向最新版本的文档。

## 架构简介

官方文档上找到了架构的流程图，附上。

{% asset_img arch2.png %}

长话短说，以下几点是关键核心。

- Item, Pipeline, Spider 是用户需要定义的部分。
- Spider 中的 parse 方法是解析过程的发动机，如果有多种页面和 Item 类型，也可以定义更多签名一致的 parse 方法。
- parse 方法可以 yield 请求，从而被调度\继续处理，也可 yield Item，直接进入流水线然后作为结果被保存。

## 命令行工具

在我的机器上安装为

```bash
/usr/bin/scrapy
```

的命令行入口是脚手架及管理工具，我打算从这里开始分析。

这个文件也是一段 Python 脚本，内容非常简短，如下：

```python
#!/usr/bin/python

# -*- coding: utf-8 -*-
import re
import sys

from scrapy.cmdline import execute

if __name__ == '__main__':
    sys.argv[0] = re.sub(r'(-script\.pyw?|\.exe)?$', '', sys.argv[0])
    sys.exit(execute())
```

可见此入口仅仅在处理了命令行参数，修整文件名之后调用了 `scrapy.cmdline.execute`方法，这个模块的路径为`/usr/lib/python3.6/site-packages/scrapy/cmdline.py`

execute 方法为全局方法，不属于某个类，摘录如下：

```python
def execute(argv=None, settings=None):
    if argv is None:
        argv = sys.argv

    if settings is None:
        settings = get_project_settings()
        # set EDITOR from environment if available
        try:
            editor = os.environ['EDITOR']
        except KeyError: pass
        else:
            settings['EDITOR'] = editor
    check_deprecated_settings(settings)

    inproject = inside_project()
    cmds = _get_commands_dict(settings, inproject)
    cmdname = _pop_command_name(argv)
    parser = optparse.OptionParser(formatter=optparse.TitledHelpFormatter(), \
        conflict_handler='resolve')
    if not cmdname:
        _print_commands(settings, inproject)
        sys.exit(0)
    elif cmdname not in cmds:
        _print_unknown_command(settings, cmdname, inproject)
        sys.exit(2)

    cmd = cmds[cmdname]
    parser.usage = "scrapy %s %s" % (cmdname, cmd.syntax())
    parser.description = cmd.long_desc()
    settings.setdict(cmd.default_settings, priority='command')
    cmd.settings = settings
    cmd.add_options(parser)
    opts, args = parser.parse_args(args=argv[1:])
    _run_print_help(parser, cmd.process_options, args, opts)

    cmd.crawler_process = CrawlerProcess(settings)
    _run_print_help(parser, _run_command, cmd, args, opts)
    sys.exit(cmd.exitcode)
```

作为命令行工具的实际入口被调用，此时没有传入任何参数到方法中。命令行参数与系统默认设置被加载。

此方法随后判断工作目录是否为具有爬虫项目的目录，从而决定是否加载额外而最为关键的几个命令如`crawl` 。`_run_print_help`方法是一个 wrapper，尝试调用后面传入的方法与参数。从`cmds`字典中获得的`cmd`是属于`ScrapyCommand`的子类型的对象。`CrawlerProcess`对象可以接管 Ctrl-C 信号等等，`_run_command`方法会判断是否需要性能分析，然后调用`cmd.run`方法。这一段代码主要都是工程实现上的封装，到这里即将可以看到实际开始工作的部分。下面一段我们查看`crawl`命令的工作流程。

## crawl 过程

crawl 命令是实际运行爬虫的命令。也是最核心的命令。希望本段的分析可以给引擎的调度运行过程一瞥。

在文件`/usr/lib/python3.6/site-packages/scrapy/commands/__init__.py`中定义了`ScrapyCommand`类，作为所有命令的抽象基类。源代码如下：

```python
class ScrapyCommand(object):

    requires_project = False
    crawler_process = None

    # default settings to be used for this command instead of global defaults
    default_settings = {}

    exitcode = 0

    def __init__(self):
        self.settings = None  # set in scrapy.cmdline

    def set_crawler(self, crawler):
        assert not hasattr(self, '_crawler'), "crawler already set"
        self._crawler = crawler

    def run(self, args, opts):
        """
        Entry point for running commands
        """
        raise NotImplementedError
```

这个类的核心是`crawler_process`字段与`run`方法。下面看 crawl 命令的 Command 类实现。

```python
class Command(ScrapyCommand):

    requires_project = True

    def syntax(self):
        return "[options] <spider>"

    def short_desc(self):
        return "Run a spider"

    def run(self, args, opts):
        if len(args) < 1:
            raise UsageError()
        elif len(args) > 1:
            raise UsageError("running 'scrapy crawl' with more than one spider is no longer supported")
        spname = args[0]

        self.crawler_process.crawl(spname, **opts.spargs)
        self.crawler_process.start()
```

重载的`run`方法调用了`crawler_process.crawl`方法。下面给出 CrawlerProcess 类的定义。

```python
class CrawlerProcess(CrawlerRunner):
    """
    A class to run multiple scrapy crawlers in a process simultaneously.

    This class extends :class:`~scrapy.crawler.CrawlerRunner` by adding support
    for starting a Twisted `reactor`_ and handling shutdown signals, like the
    keyboard interrupt command Ctrl-C. It also configures top-level logging.
    """

    def __init__(self, settings=None):
        super(CrawlerProcess, self).__init__(settings)
        install_shutdown_handlers(self._signal_shutdown)
        configure_logging(self.settings)
        log_scrapy_info(self.settings)

    def _signal_shutdown(self, signum, _):
        install_shutdown_handlers(self._signal_kill)
        signame = signal_names[signum]
        logger.info("Received %(signame)s, shutting down gracefully. Send again to force ",
                    {'signame': signame})
        reactor.callFromThread(self._graceful_stop_reactor)

    def _signal_kill(self, signum, _):
        install_shutdown_handlers(signal.SIG_IGN)
        signame = signal_names[signum]
        logger.info('Received %(signame)s twice, forcing unclean shutdown',
                    {'signame': signame})
        reactor.callFromThread(self._stop_reactor)

    def start(self, stop_after_crawl=True):
        """
        This method starts a Twisted `reactor`_, adjusts its pool size to
        :setting:`REACTOR_THREADPOOL_MAXSIZE`, and installs a DNS cache based
        on :setting:`DNSCACHE_ENABLED` and :setting:`DNSCACHE_SIZE`.

        If `stop_after_crawl` is True, the reactor will be stopped after all
        crawlers have finished, using :meth:`join`.

        :param boolean stop_after_crawl: stop or not the reactor when all
            crawlers have finished
        """
        if stop_after_crawl:
            d = self.join()
            # Don't start the reactor if the deferreds are already fired
            if d.called:
                return
            d.addBoth(self._stop_reactor)

        reactor.installResolver(self._get_dns_resolver())
        tp = reactor.getThreadPool()
        tp.adjustPoolsize(maxthreads=self.settings.getint('REACTOR_THREADPOOL_MAXSIZE'))
        reactor.addSystemEventTrigger('before', 'shutdown', self.stop)
        reactor.run(installSignalHandlers=False)  # blocking call

    def _graceful_stop_reactor(self):
        d = self.stop()
        d.addBoth(self._stop_reactor)
        return d

    def _stop_reactor(self, _=None):
        try:
            reactor.stop()
        except RuntimeError:  # raised if already stopped or in shutdown stage
            pass
```

可见这个类当中并不包含`crawl`方法。于是寻找基类，幸运地找到了这个方法。

```python
class CrawlerRunner(object):
    """
    This is a convenient helper class that keeps track of, manages and runs
    crawlers inside an already setup Twisted `reactor`_.
    """

    crawlers = property(
        lambda self: self._crawlers,
        doc="Set of :class:`crawlers <scrapy.crawler.Crawler>` started by "
            ":meth:`crawl` and managed by this class."
    )

    def __init__(self, settings=None):
        if isinstance(settings, dict) or settings is None:
            settings = Settings(settings)
        self.settings = settings
        self.spider_loader = _get_spider_loader(settings)
        self._crawlers = set()
        self._active = set()

    @property
    def spiders(self):
        warnings.warn("CrawlerRunner.spiders attribute is renamed to "
                      "CrawlerRunner.spider_loader.",
                      category=ScrapyDeprecationWarning, stacklevel=2)
        return self.spider_loader

    def crawl(self, crawler_or_spidercls, *args, **kwargs):
        """
        Run a crawler with the provided arguments.

        It will call the given Crawler's :meth:`~Crawler.crawl` method, while
        keeping track of it so it can be stopped later.

        If `crawler_or_spidercls` isn't a :class:`~scrapy.crawler.Crawler`
        instance, this method will try to create one using this parameter as
        the spider class given to it.

        Returns a deferred that is fired when the crawling is finished.

        :param crawler_or_spidercls: already created crawler, or a spider class
            or spider's name inside the project to create it
        :type crawler_or_spidercls: :class:`~scrapy.crawler.Crawler` instance,
            :class:`~scrapy.spiders.Spider` subclass or string

        :param list args: arguments to initialize the spider

        :param dict kwargs: keyword arguments to initialize the spider
        """
        crawler = self.create_crawler(crawler_or_spidercls)
        return self._crawl(crawler, *args, **kwargs)

    def _crawl(self, crawler, *args, **kwargs):
        self.crawlers.add(crawler)
        d = crawler.crawl(*args, **kwargs)
        self._active.add(d)

        def _done(result):
            self.crawlers.discard(crawler)
            self._active.discard(d)
            return result

        return d.addBoth(_done)

    def create_crawler(self, crawler_or_spidercls):
        """
        Return a :class:`~scrapy.crawler.Crawler` object.

        * If `crawler_or_spidercls` is a Crawler, it is returned as-is.
        * If `crawler_or_spidercls` is a Spider subclass, a new Crawler
          is constructed for it.
        * If `crawler_or_spidercls` is a string, this function finds
          a spider with this name in a Scrapy project (using spider loader),
          then creates a Crawler instance for it.
        """
        if isinstance(crawler_or_spidercls, Crawler):
            return crawler_or_spidercls
        return self._create_crawler(crawler_or_spidercls)

    def _create_crawler(self, spidercls):
        if isinstance(spidercls, six.string_types):
            spidercls = self.spider_loader.load(spidercls)
        return Crawler(spidercls, self.settings)

    def stop(self):
        """
        Stops simultaneously all the crawling jobs taking place.

        Returns a deferred that is fired when they all have ended.
        """
        return defer.DeferredList([c.stop() for c in list(self.crawlers)])

    @defer.inlineCallbacks
    def join(self):
        """
        join()

        Returns a deferred that is fired when all managed :attr:`crawlers` have
        completed their executions.
        """
        while self._active:
            yield defer.DeferredList(self._active)
```

`~scrapy.crawler.Crawler`为 Crawler 的类型。注意这个框架下 Crawler 与 Spider 是两个不同的概念。得益于 Python 的动态特性，我们看到代码当中可以通过字符串来动态寻找并且实例化一个 Crawler，并且调用其 crawl 方法。总之我们先跟进这个类当中一探究竟。

```python
class Crawler(object):

    def __init__(self, spidercls, settings=None):
        if isinstance(settings, dict) or settings is None:
            settings = Settings(settings)

        self.spidercls = spidercls
        self.settings = settings.copy()
        self.spidercls.update_settings(self.settings)

        self.settings.freeze()
        self.crawling = False
        self.spider = None
        self.engine = None

    @property
    def spiders(self):
            self._spiders = _get_spider_loader(self.settings.frozencopy())
        return self._spiders

    @defer.inlineCallbacks
    def crawl(self, *args, **kwargs):
        assert not self.crawling, "Crawling already taking place"
        self.crawling = True

        try:
            self.spider = self._create_spider(*args, **kwargs)
            self.engine = self._create_engine()
            start_requests = iter(self.spider.start_requests())
            yield self.engine.open_spider(self.spider, start_requests)
            yield defer.maybeDeferred(self.engine.start)
        except Exception:
            self.crawling = False
            if self.engine is not None:
                yield self.engine.close()
            raise

    def _create_spider(self, *args, **kwargs):
        return self.spidercls.from_crawler(self, *args, **kwargs)

    def _create_engine(self):
        return ExecutionEngine(self, lambda _: self.stop())

    @defer.inlineCallbacks
    def stop(self):
        if self.crawling:
            self.crawling = False
            yield defer.maybeDeferred(self.engine.stop)
```

在这里先提一下，`@defer.inlineCallbacks`是 twisted 网络库当中使用的异步封装。Scrapy 框架就是全盘构建于 twisted 之上的。对于了解 async/await 风格异步编程的读者来说，这个 attribute 相当于 async 声明，随后方法中的 yield 大致相当于 await 运算符。如有疑问，请读者自行查阅相关内容。

这里 crawl 方法当中创建了 spider 与引擎，并获取了起始 url 列表，启动引擎开始了爬取。使用过 Scrapy 框架的人所了解的那个起始 url 列表就从这里开始参与了工作。从这里开始，我们将看到我们所编写的代码是如何与框架进行交互的。



--- 未完待续 ---



## parse 方法

这个方法个人感觉是整个框架使用过程的核心。在每个相同签名的方法当中既可以迭代返回请求也可以是最终的物品，本段将试图阐释这一机制的运作原理。

## 总结

