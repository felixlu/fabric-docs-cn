===================================
环境字典 ``env``
===================================

一个简单但又是 Fabric 组成部分的概念是“环境”：一个 Python 字典子类，被作为组合设置的注册表，和内部任务的共享数据命名空间。

环境字典目前被实现为一个全局单例 ``fabric.state.env``，为方便起见，也被包含在 ``fabric.api`` 中。``env`` 中的键也经常被称为“环境变量”。

环境与配置
============================

Fabric 的大部分行为可以通过修改 ``env`` 变量来控制，例如 ``env.hosts`` （已经在 :ref:`入门导览 <defining-connections>` 中见过）。其他经常需要修改的环境变量包括：

* ``user``：Fabric 默认使用你本地用户名去建立 SSH 连接，但如果有必要，你可以用 ``env.user`` 来覆写它。文档 :doc:`execution` 部分也有关于如何针对每个主机设置用户名的信息。
* ``password``：用来显式设置你的默认连接或 sudo 密码。如果没有设置密码或设置了不正确的密码，Fabric 将会提示你输入。
* ``warn_only``：一个布尔值设置，用来表明 Fabric 是否在检测到远程错误时退出。查看 :doc:`execution` 以了解更多关于此行为的信息。

还有许多环境变量，可以查看本文档末尾的 :ref:`env-vars` 完整列表。


`~fabric.context_managers.settings` 上下文管理器
-------------------------------------------------------

在很多情况下，临时修改 ``env`` 变量以使某些指定的设置只应用到部分代码块是很有用的。Fabric 提供了一个 `~fabric.context_managers.settings` 上下文管理器，它可以接受一个或多个键/值对参数，并用来修改它所包裹的代码块范围内的 ``env``。

例如，很多情况下，设置 ``warn_only`` 是很有用的（见下文）。要将它应用到几行代码中，可以用 ``settings(warn_only=True)``。正如下面这个简化版的 ``contrib`` `~fabric.contrib.files.exists` 函数::

    from fabric.api import settings, run

    def exists(path):
        with settings(warn_only=True):
            return run('test -e %s' % path)

查看 :doc:`../api/core/context_managers` API 文档以了解关于 `~fabric.context_managers.settings` 和其他类似工具的细节。

共享状态环境
===========================

前面提到，``env`` 对象简单来说就是个字典子类，所以你的 fabfile 代码也可以在这里保存信息。有些时候，这对于在一次运行的多个任务中保持状态很有用。

.. note::

    这个 ``env`` 是很有历史的：以前的 fabfile 不是纯 Python，所以环境不是在任务间通信的唯一方式。现在，你可以直接调用其他任务或子路径，并保持模块级别的共享状态。

    在未来的版本，Fabric 将变得线程安全，在这点上，``env`` 将可能会是保持全局状态的唯一简单/安全的方式。

其他考虑
====================

在继承 ``dict`` 的同时，Fabric 的 ``env`` 也作了些修改，以使它的值可以通过属性访问的方式来读/写，这在前文也有所见。换句话来说，``env.host_string`` 和 ``env['host_string']`` 的作用是完全一样的。我们感觉属性访问经常可以节省一些打字，并使代码的可读性更高，所以这也是与 ``env`` 交互的推荐方式。

它是个字典的事实也在其他方面很有用，例如用 Python 的基于 ``dict`` 的字符串替代法，可以在你需要在一个字符串中插入多个环境变量值的时候显得尤其方便。使用“普通”的字符串替代法可能就像这样::

    print("Executing on %s as %s" % (env.host, env.user))

使用字典风格的字符串替代法就更加可读而且简洁::

        print("Executing on %(host)s as %(user)s" % env)

.. _env-vars:

环境变量完整列表
=====================

以下是所有预定义（或在 Fabric 运行时自己定义）的环境变量的完整列表。它们中的大部分都可以直接操作，但最好还是使用 `~fabric.context_managers`，可以通过 `~fabric.context_managers.settings` 或特定的上下文管理器，如 `~fabric.context_managers.cd`。

需注意的是它们中的大部分可以通过 ``fab`` 的命令行参数来设置，更多细节请参考 :doc:`fab`。在下文相应的地方也提供了交叉引用链接。

.. seealso:: :option:`--set`

.. _abort-exception:

``abort_exception``
-------------------

**默认值：** ``None``

正常情况下，Fabric 执行放弃操作的步骤是先将错误信息打印到标准错误输出，然后调用 ``sys.exit(1)``。这个设置允许你覆写这个默认行为（即 ``env.abort_exception`` 为 ``None`` 时发生的事）。

给它一个可调用的对象，它可以接受一个字符串（原来将被打印的错误信息），并返回一个异常实例。这个异常对象将被抛出，以代替（原来的 ``sys.exit`` 执行的） ``SystemExit``。

大部分情况下，你可以简单地将它设置为一个异常类，因为它完美符合了上面的描述（可调用、接受一个字符串、返回一个异常实例），例如 ``env.abort_exception = MyExceptionClass``。

.. _abort-on-prompts:

``abort_on_prompts``
--------------------

**默认值：** ``False``

当这个值为 ``True`` 时，Fabric 将以非交互模式运行。此模式下，任何需要提示用户输入（如提示输入密码、询问连接到哪个主机、fabfile 中触发的 `~fabric.operations.prompt` 等等）的时候，都会调用 `~fabric.utils.abort`。这就允许用户确保 Fabric 会话总是清楚地中止，而不是在某些预料之外的情况发生时，仍傻傻地一直在等待用户输入。

.. versionadded:: 1.1
.. seealso:: :option:`--abort-on-prompts`


``all_hosts``
-------------

**默认值：** ``[]``

由 ``fab`` 设置的当前正在执行的命令的完整主机列表。仅供显示信息。

.. seealso:: :doc:`execution`

.. _always-use-pty:

``always_use_pty``
------------------

**默认值：** ``True``

当设置为 ``False`` 时，会使 `~fabric.operations.run`/`~fabric.operations.sudo` 的行为像它们被用 ``pty=False`` 参数调用时一样。

.. seealso:: :option:`--no-pty`
.. versionadded:: 1.0

.. _colorize-errors:

``colorize_errors``
-------------------

**默认值：** ``False``

当被设置为 ``True`` 时，输出到终端的错误信息会显示成红色，警告信息则显示为洋红色，以使它们更容易被看见。

.. versionadded:: 1.7

.. _combine-stderr:

``combine_stderr``
------------------

**默认值：**: ``True``

使 SSH 层合并远程程序的 stdout 和 stderr 流输出，以避免它们在打印时混在一起。查看 :ref:`combine_streams` 以了解为什么需要这个功能，及它的效果是怎样的。

.. versionadded:: 1.0

``command``
-----------

**默认值：** ``None``

由 ``fab`` 设置的当前正在执行的命令名称（例如，执行 ``$ fab task1 task2`` 命令，当执行 ``task1`` 时， ``env.command`` 会被设置为 ``"task1"`` ，然后设置为 ``"task2"`` ）。仅供显示信息。

.. seealso:: :doc:`execution`

``command_prefixes``
--------------------

**默认值：** ``[]``

由 `~fabric.context_managers.prefix` 修改，并附加在由 `~fabric.operations.run`/`~fabric.operations.sudo` 执行的命令前面。

.. versionadded:: 1.0

.. _command-timeout:

``command_timeout``
-------------------

**默认值：** ``None``

远程命令的超时时间，单位为秒。

.. versionadded:: 1.6
.. seealso:: :option:`--command-timeout`

.. _connection-attempts:

``connection_attempts``
-----------------------

**默认值：** ``1``

Fabric 连接一台新服务器的重试次数。出于向后兼容的原因，它的默认值是只尝试连接一次。

.. versionadded:: 1.4
.. seealso:: :option:`--connection-attempts`, :ref:`timeout`

``cwd``
-------

**默认值：** ``''``

当前工作目录，用来为 `~fabric.context_managers.cd` 上下文管理器保持状态。

.. _dedupe_hosts:

``dedupe_hosts``
----------------

**默认值：** ``True``

去除合并后的主机列表中的重复项，以使任一个主机串只出现一次（例如，当使用 ``@hosts`` + ``@roles`` ，或 ``-H`` 和
``-R`` 的组合的时候）。

当被设置为 ``False`` ，就不会去除重复项，这将允许用户显式地在同一台主机上将一个任务（以串行或并行方式）运行多次。

.. versionadded:: 1.5

.. _disable-known-hosts:

``disable_known_hosts``
-----------------------

**默认值：** ``False``

如果为 ``True`` SSH 层会跳过用户的 know-hosts 文件不加载。这样可以有效地避免当一个“已知主机”改变了 key、但仍然有效（云服务器，例如 EC2）时的异常。

.. seealso:: :option:`--disable-known-hosts <-D>`, :doc:`ssh`


.. _eagerly-disconnect:

``eagerly_disconnect``
----------------------

**默认值：** ``False``

当它为 ``True`` 时， ``fab`` 会在每个单独的任务完成后关闭连接，而不是在整个运行结束后。这可以帮助避免堆积大量无用的网络会话、或因每个进程可打开的文件或网络硬件的限制而导致问题。

.. note::
    当打开这个设置时，断开连接的信息会遍布于你的输出信息始终，而不是在最后。以后的版本可能会改进这一点。

.. _effective_roles:

``effective_roles``
-------------------

**默认值：** ``[]``

由 ``fab`` 设置的当前正在执行的命令的角色列表。仅供显示信息。

.. seealso:: :doc:`execution`

.. _exclude-hosts:

``exclude_hosts``
-----------------

**默认值：** ``[]``

指定一个主机串列表，以在 ``fab`` 执行过程中 :ref:`skipped over <exclude-hosts>` 。通常通过 :option:`--exclude-hosts/-x <-x>` 来设置。

.. versionadded:: 1.1


``fabfile``
-----------

**默认值：** ``fabfile.py``

当 ``fab`` 加载 fabfile 时查找的文件名。要指定一个特定的 fabfile 文件，需要使用该文件的完整路径。显然，不可能在 fabfile 中设置这个参数，但它可以在一个 .fabricrc 文件设置，或通过命令行参数设置。

.. seealso:: :option:`--fabfile <-f>`, :doc:`fab`


.. _gateway:

``gateway``
-----------

**默认值：** ``None``

允许通过指定主机创建 SSH 驱动的网关。它的值应该是一个普通的 Fabric 主机串，和在 :ref:`env.host_string <host_string>` 中使用的一样。当它被设置时，新创建的连接将会通过这个远程 SSH 连接到最终的目的地。

.. versionadded:: 1.5

.. seealso:: :option:`--gateway <-g>`


.. _host_string:

``host_string``
---------------

**默认值：** ``None``

定义了 Fabric 在执行 `~fabric.operations.run` 、 `~fabric.operations.put` 等命令时使用的用户/主机/端口。它可以由 ``fab`` 在与已设置的主机列表交互时设置，也可以在将 Fabric 作为一个库使用时手工设置。

.. seealso:: :doc:`execution`


.. _forward-agent:

``forward_agent``
--------------------

**默认值：** ``False``

If ``True``, enables forwarding of your local SSH agent to the remote end.

.. versionadded:: 1.4

.. seealso:: :option:`--forward-agent <-A>`


``host``
--------

**默认值：** ``None``

Set to the hostname part of ``env.host_string`` by ``fab``. For informational
purposes only.

.. _hosts:

``hosts``
---------

**默认值：** ``[]``

The global host list used when composing per-task host lists.

.. seealso:: :option:`--hosts <-H>`, :doc:`execution`

.. _keepalive:

``keepalive``
-------------

**默认值：** ``0`` (i.e. no keepalive)

An integer specifying an SSH keepalive interval to use; basically maps to the
SSH config option ``ClientAliveInterval``. Useful if you find connections are
timing out due to meddlesome network hardware or what have you.

.. seealso:: :option:`--keepalive`
.. versionadded:: 1.1


.. _key:

``key``
----------------

**默认值：** ``None``

A string, or file-like object, containing an SSH key; used during connection
authentication.

.. note::
    The most common method for using SSH keys is to set :ref:`key-filename`.

.. versionadded:: 1.7


.. _key-filename:

``key_filename``
----------------

**默认值：** ``None``

May be a string or list of strings, referencing file paths to SSH key files to
try when connecting. Passed through directly to the SSH layer. May be
set/appended to with :option:`-i`.

.. seealso:: `Paramiko's documentation for SSHClient.connect() <http://docs.paramiko.org/en/latest/api/client.html#paramiko.client.SSHClient.connect>`_

.. _env-linewise:

``linewise``
------------

**默认值：** ``False``

Forces buffering by line instead of by character/byte, typically when running
in parallel mode. May be activated via :option:`--linewise`. This option is
implied by :ref:`env.parallel <env-parallel>` -- even if ``linewise`` is False,
if ``parallel`` is True then linewise behavior will occur.

.. seealso:: :ref:`linewise-output`

.. versionadded:: 1.3


.. _local-user:

``local_user``
--------------

A read-only value containing the local system username. This is the same value
as :ref:`user`'s initial value, but whereas :ref:`user` may be altered by CLI
arguments, Python code or specific host strings, :ref:`local-user` will always
contain the same value.

.. _no_agent:

``no_agent``
------------

**默认值：** ``False``

If ``True``, will tell the SSH layer not to seek out running SSH agents when
using key-based authentication.

.. versionadded:: 0.9.1
.. seealso:: :option:`--no_agent <-a>`

.. _no_keys:

``no_keys``
------------------

**默认值：** ``False``

If ``True``, will tell the SSH layer not to load any private key files from
one's ``$HOME/.ssh/`` folder. (Key files explicitly loaded via ``fab -i`` will
still be used, of course.)

.. versionadded:: 0.9.1
.. seealso:: :option:`-k`

.. _env-parallel:

``parallel``
-------------------

**默认值：** ``False``

When ``True``, forces all tasks to run in parallel. Implies :ref:`env.linewise
<env-linewise>`.

.. versionadded:: 1.3
.. seealso:: :option:`--parallel <-P>`, :doc:`parallel`

.. _password:

``password``
------------

**默认值：** ``None``

The default password used by the SSH layer when connecting to remote hosts,
**and/or** when answering `~fabric.operations.sudo` prompts.

.. seealso:: :option:`--initial-password-prompt <-I>`, :ref:`env.passwords <passwords>`, :ref:`password-management`

.. _passwords:

``passwords``
-------------

**默认值：** ``{}``

This dictionary is largely for internal use, and is filled automatically as a
per-host-string password cache. Keys are full :ref:`host strings
<host-strings>` and values are passwords (strings).

.. seealso:: :ref:`password-management`

.. versionadded:: 1.0


.. _env-path:

``path``
--------

**默认值：** ``''``

Used to set the ``$PATH`` shell environment variable when executing commands in
`~fabric.operations.run`/`~fabric.operations.sudo`/`~fabric.operations.local`.
It is recommended to use the `~fabric.context_managers.path` context manager
for managing this value instead of setting it directly.

.. versionadded:: 1.0


.. _pool-size:

``pool_size``
-------------

**默认值：** ``0``

Sets the number of concurrent processes to use when executing tasks in parallel.

.. versionadded:: 1.3
.. seealso:: :option:`--pool-size <-z>`, :doc:`parallel`

.. _prompts:

``prompts``
-------------

**默认值：** ``{}``

The ``prompts`` dictionary allows users to control interactive prompts. If a
key in the dictionary is found in a command's standard output stream, Fabric
will automatically answer with the corresponding dictionary value.

.. versionadded:: 1.9

.. _port:

``port``
--------

**默认值：** ``None``

Set to the port part of ``env.host_string`` by ``fab`` when iterating over a
host list. May also be used to specify a default port.

.. _real-fabfile:

``real_fabfile``
----------------

**默认值：** ``None``

Set by ``fab`` with the path to the fabfile it has loaded up, if it got that
far. For informational purposes only.

.. seealso:: :doc:`fab`


.. _remote-interrupt:

``remote_interrupt``
--------------------

**默认值：** ``None``

Controls whether Ctrl-C triggers an interrupt remotely or is captured locally,
as follows:

* ``None`` (the default): only `~fabric.operations.open_shell` will exhibit
  remote interrupt behavior, and
  `~fabric.operations.run`/`~fabric.operations.sudo` will capture interrupts
  locally.
* ``False``: even `~fabric.operations.open_shell` captures locally.
* ``True``: all functions will send the interrupt to the remote end.

.. versionadded:: 1.6


.. _rcfile:

``rcfile``
----------

**默认值：** ``$HOME/.fabricrc``

Path used when loading Fabric's local settings file.

.. seealso:: :option:`--config <-c>`, :doc:`fab`

.. _reject-unknown-hosts:

``reject_unknown_hosts``
------------------------

**默认值：** ``False``

If ``True``, the SSH layer will raise an exception when connecting to hosts not
listed in the user's known-hosts file.

.. seealso:: :option:`--reject-unknown-hosts <-r>`, :doc:`ssh`

.. _system-known-hosts:

``system_known_hosts``
------------------------

**默认值：** ``None``

If set, should be the path to a :file:`known_hosts` file.  The SSH layer will
read this file before reading the user's known-hosts file.

.. seealso:: :doc:`ssh`

``roledefs``
------------

**默认值：** ``{}``

Dictionary defining role name to host list mappings.

.. seealso:: :doc:`execution`

.. _roles:

``roles``
---------

**默认值：** ``[]``

The global role list used when composing per-task host lists.

.. seealso:: :option:`--roles <-R>`, :doc:`execution`

.. _shell:

``shell``
---------

**默认值：** ``/bin/bash -l -c``

Value used as shell wrapper when executing commands with e.g.
`~fabric.operations.run`. Must be able to exist in the form ``<env.shell>
"<command goes here>"`` -- e.g. the default uses Bash's ``-c`` option which
takes a command string as its value.

.. seealso:: :option:`--shell <-s>`,
             :ref:`FAQ on bash as default shell <faq-bash>`, :doc:`execution`

.. _skip-bad-hosts:

``skip_bad_hosts``
------------------

**默认值：** ``False``

If ``True``, causes ``fab`` (or non-``fab`` use of `~fabric.tasks.execute`) to skip over hosts it can't connect to.

.. versionadded:: 1.4
.. seealso::
    :option:`--skip-bad-hosts`, :ref:`excluding-hosts`, :doc:`execution`


.. _ssh-config-path:

``ssh_config_path``
-------------------

**默认值：** ``$HOME/.ssh/config``

Allows specification of an alternate SSH configuration file path.

.. versionadded:: 1.4
.. seealso:: :option:`--ssh-config-path`, :ref:`ssh-config`

``ok_ret_codes``
------------------------

**默认值：** ``[0]``

Return codes in this list are used to determine whether calls to
`~fabric.operations.run`/`~fabric.operations.sudo`/`~fabric.operations.sudo`
are considered successful.

.. versionadded:: 1.6

.. _sudo_prefix:

``sudo_prefix``
---------------

**默认值：** ``"sudo -S -p '%(sudo_prompt)s' " % env``

The actual ``sudo`` command prefixed onto `~fabric.operations.sudo` calls'
command strings. Users who do not have ``sudo`` on their default remote
``$PATH``, or who need to make other changes (such as removing the ``-p`` when
passwordless sudo is in effect) may find changing this useful.

.. seealso::

    The `~fabric.operations.sudo` operation; :ref:`env.sudo_prompt
    <sudo_prompt>`

.. _sudo_prompt:

``sudo_prompt``
---------------

**默认值：** ``"sudo password:"``

Passed to the ``sudo`` program on remote systems so that Fabric may correctly
identify its password prompt.

.. seealso::

    The `~fabric.operations.sudo` operation; :ref:`env.sudo_prefix
    <sudo_prefix>`

.. _sudo_user:

``sudo_user``
-------------

**默认值：** ``None``

Used as a fallback value for `~fabric.operations.sudo`'s ``user`` argument if
none is given. Useful in combination with `~fabric.context_managers.settings`.

.. seealso:: `~fabric.operations.sudo`

.. _env-tasks:

``tasks``
-------------

**默认值：** ``[]``

Set by ``fab`` to the full tasks list to be executed for the currently
executing command. For informational purposes only.

.. seealso:: :doc:`execution`

.. _timeout:

``timeout``
-----------

**默认值：** ``10``

Network connection timeout, in seconds.

.. versionadded:: 1.4
.. seealso:: :option:`--timeout`, :ref:`connection-attempts`

``use_shell``
-------------

**默认值：** ``True``

Global setting which acts like the ``shell`` argument to
`~fabric.operations.run`/`~fabric.operations.sudo`: if it is set to ``False``,
operations will not wrap executed commands in ``env.shell``.


.. _use-ssh-config:

``use_ssh_config``
------------------

**默认值：** ``False``

Set to ``True`` to cause Fabric to load your local SSH config file.

.. versionadded:: 1.4
.. seealso:: :ref:`ssh-config`


.. _user:

``user``
--------

**默认值：** User's local username

The username used by the SSH layer when connecting to remote hosts. May be set
globally, and will be used when not otherwise explicitly set in host strings.
However, when explicitly given in such a manner, this variable will be
temporarily overwritten with the current value -- i.e. it will always display
the user currently being connected as.

To illustrate this, a fabfile::

    from fabric.api import env, run

    env.user = 'implicit_user'
    env.hosts = ['host1', 'explicit_user@host2', 'host3']

    def print_user():
        with hide('running'):
            run('echo "%(user)s"' % env)

and its use::

    $ fab print_user

    [host1] out: implicit_user
    [explicit_user@host2] out: explicit_user
    [host3] out: implicit_user

    Done.
    Disconnecting from host1... done.
    Disconnecting from host2... done.
    Disconnecting from host3... done.

As you can see, during execution on ``host2``, ``env.user`` was set to
``"explicit_user"``, but was restored to its previous value
(``"implicit_user"``) afterwards.

.. note::

    ``env.user`` is currently somewhat confusing (it's used for configuration
    **and** informational purposes) so expect this to change in the future --
    the informational aspect will likely be broken out into a separate env
    variable.

.. seealso:: :doc:`execution`, :option:`--user <-u>`

``version``
-----------

**默认值：** current Fabric version string

Mostly for informational purposes. Modification is not recommended, but
probably won't break anything either.

.. seealso:: :option:`--version <-V>`

.. _warn_only:

``warn_only``
-------------

**默认值：** ``False``

Specifies whether or not to warn, instead of abort, when
`~fabric.operations.run`/`~fabric.operations.sudo`/`~fabric.operations.local`
encounter error conditions.

.. seealso:: :option:`--warn-only <-w>`, :doc:`execution`
