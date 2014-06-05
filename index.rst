==================================
欢迎访问 Fabric 中文版文档！
==================================

本站覆盖了 Fabric 的用法和 API 文档。若想了解 Fabric 是什么，包括它的更新日志和如何维护该项目，请访问 `Fabric 项目官方网站 <http://fabfile.org>`_ 。

入门指导
--------

对于新用户，与/或想大概了解 Fabric 的基本功能的同学，请看 :doc:`tutorial` 。本文档的其他部分将假设你至少已经大概熟悉其中的内容。

.. toctree::
    :hidden:

    tutorial


.. _usage-docs:

用法文档
-------------------

下面的列表包含了 Fabric 散文（非 API）文档的所有主要章节。这些内容在 :doc:`tutorial` 中提到的概念基础上进行了扩展，并覆盖了一些进阶主题。

.. toctree::
    :maxdepth: 2
    :glob:

    usage/*


.. _api_docs:

API 文档
-----------------

Fabric 维护了两套 API 文档，它们都是根据代码中的 docstring 自动生成的，也十分详尽。

.. _core-api:

核心 API
~~~~~~~~

**核心** API 是指构成 Fabric 基础构建块的函数、类和方法（例如 `~fabric.operations.run` 和 `~fabric.operations.sudo`）。而其他部分（下文的“扩展 API”和用户的 fabfile）都是在这些核心 API 的基础之上构建的。

.. toctree::
    :maxdepth: 和
    :glob:

    api/core/*

.. _contrib-api:

扩展 API
~~~~~~~~~~~

Fabric 的 **扩展** 包包括常用而有用的工具（通常是从用户的 fabfile 中合并进来的），可用于用户 I/O、修改远程文件等任务中。核心 API 倾向于保持小巧、不随意变更，扩展包则会随着更多的用户案例被解决并添加进来，而不断成长进化（同时尽量保持向后兼容）。

.. toctree::
    :maxdepth: 1
    :glob:

    api/contrib/*
