=====================
入门导览
=====================

欢迎来到 Fabric！

本文档走马观花式地介绍 Fabric 特性，也是对其使用的快速指导。其他文档（这里通篇的链接都指向它们）可以在 :ref:`用法文档 <usage-docs>` 里找到——请不要忘记也看看它们。

Fabric 是什么？
===============

正如 ``README`` 所说：

    Fabric 是一个 Python (2.5-2.7) 库和命令行工具，用来流水线化执行 SSH 以部署应用或系统管理任务。

更具体地说，Fabric 是：

* 一个让你通过 **命令行** 执行 **任意 Python 函数** 的工具；
* 一个让通过 SSH 执行 Shell 命令更加 **容易** 和 **蟒样** 的子程序库（建立于一个更低层次的库）。

自然地，大部分用户把这两件事结合着用，使用 Fabric 来写和执行 Python 函数或 **任务** ，以实现与远程服务器的自动化交互。让我们先睹为快吧。

你好， ``fab``
==============

如果没有下面这个国际惯例，这个文档恐怕不能算是个合格的入门指导::

    def hello():
        print("Hello world!")

把上述代码放在你当前的工作目录中一个名为 ``fabfile.py`` 的 Python 模块文件中。然后这个 ``hello`` 函数就可以用安装 Fabric 时顺便装上的 ``fab`` 工具来执行了，它将如你所料地工作::

    $ fab hello
    Hello world!

    Done.

这就是这个模块的所有作用。这个功能让 Fabric 可以作为一个极其基本的构建工具来使用，简单到甚至不用导入它的任何 API。

.. note::

    这个 ``fab`` 简单地导入了你的 fabfile 并执行你定义的一个或多个函数。这里并没有任何魔术——任何你能在一个普通 Python 模块中做的事情都同样可以在一个 fabfile 中完成。

.. seealso:: :ref:`execution-strategy`, :doc:`/usage/tasks`, :doc:`/usage/fab`


任务参数
==============

就像你在常规的 Python 编程中那样，在执行任务时传递一些运行时参数经常能帮上大忙。Fabric 支持用兼容 Shell 的参数用法： ``<任务名>:<参数>, <关键字参数名>=<参数值>,...`` 虽然有点勉强，但可以扩展上面的例子，让它只向你 say hello::

    def hello(name="world"):
        print("Hello %s!" % name)

默认情况下，调用 ``fab hello`` 仍然会像之前那样工作，但现在我们可以做些个性化定制了::

    $ fab hello:name=Jeff
    Hello Jeff!

    Done.

用过 Python 编程的同学可能已经猜到了，这样调用也是完全一样的::

    $ fab hello:Jeff
    Hello Jeff!

    Done.

目前，参数值只能作为 Python 字符串来使用，如果要使用复杂类型，例如列表，会需要一些字符串操作处理。将来的版本可能会添加一个类型转换系统，以简化这类处理。

.. seealso:: :ref:`task-arguments`

本地命令
==============

在前面的例子中， ``fab`` 实际上只节省了数行 ``if __name__ == "__main__"`` 这样的固定样板代码而已。Fabric 更多地被设计为使用它自己的 API，它们包括执行 Shell 命令、传送文件等等的函数（或 **操作** ）。

我们以一个 Web 应用为例来创建一个 fabfile。具体的情景如下：这个 Web 应用用一台远程服务器 ``vcshost`` 上的 Git 管理代码，我们把它的代码库克隆到了本地 ``localhost`` 中。当我们把修改后的代码 push 回 ``vcshost`` 的时候，我们想自动就立即把新的版本安装到另一台远程服务器 ``my_server`` 上。我们将用自动化的本地和远程 Git 命令来完成这些工作。

fabfile 文件最好放在一个项目的根目录::

    .
    |-- __init__.py
    |-- app.wsgi
    |-- fabfile.py <-- our fabfile!
    |-- manage.py
    `-- my_app
        |-- __init__.py
        |-- models.py
        |-- templates
        |   `-- index.html
        |-- tests.py
        |-- urls.py
        `-- views.py

.. note::

    我们在这里用的是一个 Django 应用，但这仅仅是个例子——Fabric 并未与任何外部代码绑定，除了它的 SSH 库。

作为起步，可能我们希望先执行测试，然后再提交到 VCS（版本控制系统），为部署作好准备::

    from fabric.api import local

    def prepare_deploy():
        local("./manage.py test my_app")
        local("git add -p && git commit")
        local("git push")

这段代码的输出会是这样::

    $ fab prepare_deploy
    [localhost] run: ./manage.py test my_app
    Creating test database...
    Creating tables
    Creating indexes
    ..........................................
    ----------------------------------------------------------------------
    Ran 42 tests in 9.138s

    OK
    Destroying test database...

    [localhost] run: git add -p && git commit

    <interactive Git add / git commit edit message session>

    [localhost] run: git push

    <git push session, possibly merging conflicts interactively>

    Done.

这段代码很简单，导入一个 Fabric API： `~fabric.operations.local` ，然后用它执行本地 Shell 命令并与之交互，剩下的 Fabric API 也是类似的——它们都只是 Python 而已。

.. seealso:: :doc:`api/core/operations`, :ref:`fabfile-discovery`

用你的方式来组织
====================

因为 Fabric“只是 Python”，你可以以你想要的任何方式来组织你的 fabfile。例如，把任务分割成多个子任务::

    from fabric.api import local

    def test():
        local("./manage.py test my_app")

    def commit():
        local("git add -p && git commit")

    def push():
        local("git push")

    def prepare_deploy():
        test()
        commit()
        push()

这个 ``prepare_deploy`` 任务仍可以像之前那样调用，但现在只要你想，就可以调用更细粒度的子任务了。

故障
=======

我们的基本案例已经可以正常工作了，但如果测试失败了会发生什么事？没准我们想来个急刹车，并在部署之前修复这些失败的测试。

Fabric 会检查被调用程序的返回值，如果这些程序没有干净地退出，Fabric 会放弃操作。下面我们就来看看如果一个测试用例遇到错误时会发生什么事::

    $ fab prepare_deploy
    [localhost] run: ./manage.py test my_app
    Creating test database...
    Creating tables
    Creating indexes
    .............E............................
    ======================================================================
    ERROR: testSomething (my_project.my_app.tests.MainTests)
    ----------------------------------------------------------------------
    Traceback (most recent call last):
    [...]

    ----------------------------------------------------------------------
    Ran 42 tests in 9.138s

    FAILED (errors=1)
    Destroying test database...

    Fatal error: local() encountered an error (return code 2) while executing './manage.py test my_app'

    Aborting.

太好了！我们什么都不用做，Fabric 检测到了错误并放弃了操作，不会继续执行 ``commit`` 任务。

.. seealso:: :ref:`故障处理（用法文档） <failures>`

故障处理
----------------

但如果我们想更加灵活，给用户另一个选择，又该怎么办呢？一个名为 :ref:`warn_only` 的设置（或 **环境变量** ，经常缩写为 **env var** ）可以把放弃变成警告，使得灵活处理错误成为现实。

让我们把这个设置丢到我们的 ``test`` 函数中，然后看看这个 `~fabric.operations.local` 调用的结果如何::

    from __future__ import with_statement
    from fabric.api import local, settings, abort
    from fabric.contrib.console import confirm

    def test():
        with settings(warn_only=True):
            result = local('./manage.py test my_app', capture=True)
        if result.failed and not confirm("Tests failed. Continue anyway?"):
            abort("Aborting at user request.")

    [...]

在添加这个新特性时，我们引入了一些新东西：

* 在 Python 2.5 中，需要导入 ``__future__`` 才能使用 ``with`` ；
* Fabric 的 `contrib.console <fabric.contrib.console>` 子模块包含了 `~fabric.contrib.console.confirm` 函数，用来做简单的 yes/no 提示；
* 上下文管理器 `~fabric.context_managers.settings` 用来将设置应用到某个特定的代码块中；
* 运行命令的操作，如 `~fabric.operations.local` ，可以返回一个包含该操作的结果信息（例如 ``.failed`` 或 ``.return_code`` ）的对象；
* 还有 `~fabric.utils.abort` 函数，可以用来手工取消执行。

然而，即使在增加了上述复杂度之后，整个处理过程仍然很容易理解，而且它已经远比之前灵活了。

.. seealso:: :doc:`api/core/context_managers`, :ref:`env-vars`

建立连接
==================

我们开始让 fabfile 回到主旨吧：定义一个 ``deploy`` 任务，让它在一台或多台远程服务器上运行，并保证代码是最新的::

    def deploy():
        code_dir = '/srv/django/myproject'
        with cd(code_dir):
            run("git pull")
            run("touch app.wsgi")

这里再次引入了一些新的概念：

* Fabric 就是 Python——所以我们可以自由地使用变量、字符串等常规的 Python 代码结构；
* `~fabric.context_managers.cd` 是一个很方便的前缀命令，相当于执行 ``cd /to/some/directory`` 命令，这个命令和 `~fabric.context_managers.lcd` 一样，不过后者针对本地；
* `~fabric.operations.run` 则和 `~fabric.operations.local` 类似，但它是运行 **远程** 命令而非本地。

我们还需要确认在文件顶部导入了新的函数::

    from __future__ import with_statement
    from fabric.api import local, settings, abort, run, cd
    from fabric.contrib.console import confirm

改好之后，我们再来部署::

    $ fab deploy
    No hosts found. Please specify (single) host string for connection: my_server
    [my_server] run: git pull
    [my_server] out: Already up-to-date.
    [my_server] out:
    [my_server] run: touch app.wsgi

    Done.

我们从来没有在 fabfile 中指定任何连接信息，所以 Fabric 不知道该在哪里运行那些远程命令。当遇到这种情况，Fabric 会在运行时提示我们。连接的定义使用 SSH 风格的“主机串”（例如： ``user@host:port`` ），默认使用你的本地用户名——所以在这个例子中，我们只需要指定主机名 ``my_server`` 。

远程交互
--------------------

如果你已经签出过代码， ``git pull`` 就能很好地工作——但如果这是第一次部署呢？如果还能用 ``git clone`` 来处理这种情况那才叫棒呢::

    def deploy():
        code_dir = '/srv/django/myproject'
        with settings(warn_only=True):
            if run("test -d %s" % code_dir).failed:
                run("git clone user@vcshost:/path/to/repo/.git %s" % code_dir)
        with cd(code_dir):
            run("git pull")
            run("touch app.wsgi")

就像我们上面调用 `~fabric.operations.local` 一样， `~fabric.operations.run` 也让我们基于 Shell 命令构建干净的 Python 层逻辑。然后这里最有趣的部分是 ``git clone`` ：因为我们是用 Git 的 SSH 方法来访问 Git 服务器上的代码库，这意味着我们的远程 `~fabric.operations.run` 调用本身需要身份验证。

旧版本的 Fabric（和其他类似的高层次 SSH 库）像在监狱里一样运行远程命令，无法在本地交互。当你很迫切需要输入密码或与远程程序交互时，这就很成问题。

Fabric 1.0 和后续的版本突破了这个限制，并保证你总是能和另一边对话。让我们看看当我们在一台没有 Git checkout 的新服务器上运行更新后的 ``deploy`` 任务时会发生什么::

    $ fab deploy
    No hosts found. Please specify (single) host string for connection: my_server
    [my_server] run: test -d /srv/django/myproject

    Warning: run() encountered an error (return code 1) while executing 'test -d /srv/django/myproject'

    [my_server] run: git clone user@vcshost:/path/to/repo/.git /srv/django/myproject
    [my_server] out: Cloning into /srv/django/myproject...
    [my_server] out: Password: <enter password>
    [my_server] out: remote: Counting objects: 6698, done.
    [my_server] out: remote: Compressing objects: 100% (2237/2237), done.
    [my_server] out: remote: Total 6698 (delta 4633), reused 6414 (delta 4412)
    [my_server] out: Receiving objects: 100% (6698/6698), 1.28 MiB, done.
    [my_server] out: Resolving deltas: 100% (4633/4633), done.
    [my_server] out:
    [my_server] run: git pull
    [my_server] out: Already up-to-date.
    [my_server] out:
    [my_server] run: touch app.wsgi

    Done.

注意那个 ``Password:`` 提示——那就是我们在 Web 服务器上的远程 ``git`` 调用在询问 Git 密码。我们可以在里面输入密码，然后像往常一样继续克隆。

.. seealso:: :doc:`/usage/interactivity`


.. _defining-connections:

预先定义连接
-------------------------------

在运行时输入连接信息已经落后太多了，所以 Fabric 提供了一种方便的办法，在你的 fabfile 或命令行中指定。我们不打算在这里完全展开来说，但我们会向你展示最常用的：设置全局主机列表 :ref:`env.hosts <hosts>` 。

:doc:`env <usage/env>` 是一个全局的类字典对象，驱动着 Fabric 的大部分设置，而且可以带着属性写进去（事实上，前面见过的 `~fabric.context_managers.settings` 是它的一个简单包装）。因此，我们可以在模块层次上，在 fabfile 的顶部附近修改它，就像这样::

    from __future__ import with_statement
    from fabric.api import *
    from fabric.contrib.console import confirm

    env.hosts = ['my_server']

    def test():
        do_test_stuff()

当 ``fab`` 加载我们的 fabfile 时，我们对 ``env`` 的修改将被执行，并保存为对设置的修改。最终的结果就如上面所示：我们的 ``deploy`` 任务将在 ``my_server`` 上运行。

这也是你如何告诉 Fabric 一次在多台远程服务器上运行的方法：因为 ``env.hosts`` 是一个列表， ``fab`` 对它进行迭代，为每个连接调用指定的任务。

.. seealso:: :doc:`usage/env`, :ref:`host-lists`


小结
==========

在经过了这么多，我们的完整的 fabfile 文件仍然相当短。下面是它的完整内容::

    from __future__ import with_statement
    from fabric.api import *
    from fabric.contrib.console import confirm

    env.hosts = ['my_server']

    def test():
        with settings(warn_only=True):
            result = local('./manage.py test my_app', capture=True)
        if result.failed and not confirm("Tests failed. Continue anyway?"):
            abort("Aborting at user request.")

    def commit():
        local("git add -p && git commit")

    def push():
        local("git push")

    def prepare_deploy():
        test()
        commit()
        push()

    def deploy():
        code_dir = '/srv/django/myproject'
        with settings(warn_only=True):
            if run("test -d %s" % code_dir).failed:
                run("git clone user@vcshost:/path/to/repo/.git %s" % code_dir)
        with cd(code_dir):
            run("git pull")
            run("touch app.wsgi")

这个 fabfile 使用了 Fabric 的相当一大部分特性集：

* 定义 fabfile 任务，并用 :doc:`fab <usage/fab>` 运行；
* 用 `~fabric.operations.local` 调用本地 Shell 命令；
* 用 `~fabric.context_managers.settings` 修改环境变量；
* 处理命令故障、提示用户及手工取消；
* 还有定义主机列表和以 `~fabric.operations.run` 运行远程命令。

然而，还有更多内容没有在这里覆盖。你还可以看看所有“参见”中提供的链接，和文档内容 :doc:`索引 <index>` 表。

能看到这里真不容易，谢谢！