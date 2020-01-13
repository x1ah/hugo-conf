+++
title = "Python2 迁移到 Python3 小事记"
date = "2019-11-12"
author = "x1ah"
cover = ""
tags = ["Python"]
keywords = ["Python"]
description = "公司主项目的代码有七八年的历史，二三十万行的 Python2 代码，由于越来越多库不再支持 Python2，以及开发体验，一年前打算全面开启迁移工作，至今已经慢慢开始切分一些流量到 Python3 下了，这里记录迁移过程，以及遇到的一些坑。持续更新"
showFullContent = false
+++

## 前期规划

在开始迁移前，需要大致盘点一下都会有哪些工作量，哪些代码需要做兼容，哪些服务需要做迁移。前期可以大概分成一下几部分：

- 主项目（主要的项目，承担了主要的日常开发任务以及业务需求）
- 其他服务（为主项目服务的各个服务，如支付、IM、广告等）
- 依赖库，依赖又包括：
    - 公司内部基础组件
    - 第三方依赖

在列出所有的 "代码清单" 后，需要有个先后顺序，来逐步的进行迁移。首先上面提到的三个大点中，**其他服务** 其实优先级并不高，因为日常不怎么会开发，处于维护状态。因此保持正常运行即可，优先进行其他两项的迁移。而依赖库处处在引用，不提前进行 Python3 适配其他工作将无法进行。因此适配顺序如下：

1. 排查第三方依赖库，测试，升级到兼容 Python2/Python3 的版本
2. 排查公司内部基础组件库，测试，兼容适配 Python2/Python3
3. 进行主项目的代码层面适配，使用工具和一些库进行 2 和 3 的适配，使现有代码能同时在 2 和 3 下面跑。增加 py3 环境的单元测试。

在一切开始之前，还需要保证日常新加的代码不再引入不兼容的代码，因此应该提前使用 [pre-commit](https://pre-commit.com/) 对每个 commit 进行检查，使用 [pylint](https://github.com/pycqa/pylint) 进行兼容性检查，配置如下：

```shell
# .pre-commit-config.yaml

-   repo: https://github.com/xiachufang/mirrors-pylint
    rev: v1.9.2
    hooks:
    -   id: pylint
        args:
          - --py3k
          - --score=n
```

## 迁移中

迁移办法一般是先排查关键字，如 `iteritems`/`itervalues`/`xrange` 等，这些可以全部使用 [six](https://six.readthedocs.io/) 相应方法直接替换。 除此之外，应该给单元测试增加 Python3 环境，这样首先保证单元测试能在 Python3 下跑通，在调通单元测试之后，如果测试覆盖率高，那么基本已经改完很大一部分代码了。在给代码做适配是，可以使用 [futurize](http://python-future.org/index.html) 来自动修改一些代码，减少一些重复工作。并且可以参考 futurize 的 [Cheat Sheet: Writing Python 2-3 compatible code](http://python-future.org/compatible_idioms.html) 来做对照，进行修改代码。


### 会遇到的问题

#### 关键字/方法
上面有提到，某些关键字或者方法，到了 Python3 里面已经没有了，比如 `xrange`/`dict.iteritems`/`dict.itervalues`，这个一般全局搜索就能排除掉。

#### 内置函数返回类型
- 在 Python2 中，`dict.keys` 返回的是一个 list，而到了 Python3 中，返回的是一个 `dict_keys`，如果存在使用下标取，那么是会有问题的。如 `{"K": "V"}.keys()[0]`
- Python2 中，map/reduce/filter 之类的关键字返回的是个 list，到了 3 中，返回的是 generator，如果需要下标访问是需要转成 list/tuple 的
- etc...

### str/bytes/unicode
这个应当属迁移的最繁琐的地方。

中间遇到一次问题，排查了很久。代码库中有一个 `@cache` 装饰器，用来缓存函数返回值，在 py2 中有如下一段代码:

```python

@cache('cache_key')
def foo():
    return "py2 返回结构"
```

cache 拿到函数返回值后，原样放进 Memcached，因为 Python2 中，str/bytes 实际上是不敏感的，甚至可以等同。该 `foo` 函数在 py2 下返回的是一个带 encoding 的 str，可以看做是 bytes 类型，这时候，如果在 py3 下把缓存结果取出来，那么将会是拿到一个 bytes 类型: `b"py2 返回结构"`，这里就很容易出错了，在 Python2 下时，这个和 str 一样，可以当做 str 处理，但是 Python3 必须正视类型，该用 str(text type) 就不能用 bytes。


## 代码库迁移完成

把代码库全部兼容 Python2 和 Python3 之后，这时候代码库是可以同时在 Python2 和 Python3 上跑的，因此可以逐步开始切分流量，大概可以分成这么几个步骤：
1. 把 staging 流量复制到 Python3 的环境
2. 部署 production 的 Python3 环境，并切分办公室流量至 Python3 环境
3. 切分线上小部分流量到 Python3 环境，并逐步增加，直至全部覆盖


## 迁移后工作

全部迁移完成后，自然会给代码加上 type hint，前期可以使用 pre-commit 增加 mypy 类型检查，并且只检查改动到的文件。与此同时，使用 [MonkeyType](https://github.com/Instagram/MonkeyType) 收集类型，自动添加一部分，减少工作量。

至此，迁移工作已经全部完成。
