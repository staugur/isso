Server Configuration
====================

The Isso configuration file is an `INI-style`__ textfile. It reads integers,
booleans, strings and lists. Here's the default isso configuration:
`isso.conf <https://github.com/posativ/isso/blob/master/share/isso.conf>`. A
basic configuration from scratch looks like this:

.. code-block:: ini

    [general]
    dbpath = /var/lib/isso/comments.db
    host = https://example.tld/
    [server]
    listen = http://localhost:1234/

To use your configuration file with Isso, append ``-c /path/to/cfg`` to the
executable or run Isso with an environment variable:

.. code-block:: sh

    ~> isso -c path/to/isso.cfg
    ~> env ISSO_SETTINGS=path/to/isso.cfg isso

__ https://en.wikipedia.org/wiki/INI_file

Sections covered in this document:

.. contents::
    :local:


General
-------

In this section, you configure most comment-related options such as database path,
session key and hostname. Here are the default values for this section:

.. code-block:: ini

    [general]
    dbpath = /tmp/isso.db
    name =
    host =
    max-age = 15m
    notify = stdout
    log-file =

dbpath
    file location to the SQLite3 database, highly recommended to change this
    location to a non-temporary location!

.. _pac-name:

name
    required to dispatch :ref:`multiple websites <configure-multiple-sites>`,
    not used otherwise.

.. _pac-host:

host
    Your website(s). If Isso is unable to connect to at least one site, you'll
    get a warning during startup and comments are most likely non-functional.

    You'll need at least one host/website to run Isso. This is due to security
    reasons: Isso uses CORS_ to embed comments and to restrict comments only to
    your website, you have to "whitelist" your website(s).

    I recommend the first value to be a non-SSL website that is used as fallback
    if Firefox users (and only those) supress their HTTP referer completely.

    .. code-block:: ini

        [general]
        host =
            http://example.tld/
            https://example.tld/

max-age
    time range that allows users to edit/remove their own comments. See
    :ref:`Appendum: Timedelta <appendum-timedelta>` for valid values.

.. _pac-notify:

notify
    Select notification backend(s) for new comments, separated by comma.
    Available backends:

    stdout
        Log to standard output. Default, if none selected. Note, this
        functionality is broken since a few releases.

    smtp
        Send notifications via SMTP on new comments with activation (if
        moderated) and deletion links.

    wechat
        有新评论时通过微信发送提醒（支持审核）

.. _pac-reply-notifications:

reply-notifications
    Allow users to request E-mail notifications for replies to their post.

    It is highly recommended to also turn on moderation when enabling this
    setting, as Isso can otherwise be easily exploited for sending spam.

    Do not forget to configure the client accordingly.

log-file
    Log console messages to file instead of standard out.

gravatar
    When set to ``true`` this will add the property "gravatar_image"
    containing the link to a gravatar image to every comment. If a comment
    does not contain an email address, gravatar will render a random icon.
    This is only true when using the default value for "gravatar-url"
    which contains the query string param ``d=identicon`` ...

gravatar-url
    Url for gravatar images. The "{}" is where the email hash will be placed.
    Defaults to "https://www.gravatar.com/avatar/{}?d=identicon"



.. _CORS: https://developer.mozilla.org/en/docs/HTTP/Access_control_CORS

.. _pac-moderation:

.. _configure-moderation:

Moderation
----------

Enable moderation queue and handling of comments still in moderation queue

.. code-block:: ini

    [moderation]
    enabled = false
    approve-if-email-previously-approved = false
    purge-after = 30d


enabled
    enable comment moderation queue. This option only affects new comments.
    Comments in moderation queue are not visible to other users until you
    activate them.

approve-if-email-previously-approved
    automatically approve comments by an email address if that address has
    had a comment approved within the last 6 months. No ownership verification
    is done on the entered email address. This means that if someone is able
    to guess correctly the email address used by a previously approved author,
    they will be able to have their new comment auto-approved.

purge-after
    remove unprocessed comments in moderation queue after given time.


Server
------

HTTP server configuration.

.. code-block:: ini

    [server]
    listen = http://localhost:8080
    reload = off
    profile = off

listen
    interface to listen on. Isso supports TCP/IP and unix domain sockets:

    .. code-block:: ini

        ; UNIX domain socket
        listen = unix:///tmp/isso.sock
        ; TCP/IP
        listen = http://localhost:1234/

    When ``gevent`` is available, it is automatically used for `http://`
    Currently, gevent can not handle http requests on unix domain socket
    (see `#295 <https://github.com/surfly/gevent/issues/295>`_ and
    `#299 <https://github.com/surfly/gevent/issues/299>`_ for details).

    Does not apply for `uWSGI`.

public-endpoint
    public URL that Isso is accessed from by end users. Should always be
    a http:// or https:// absolute address. If left blank, automatic
    detection is attempted. Normally only needs to be specified if
    different than the `listen` setting.

reload
    reload application, when the source code has changed. Useful for
    development. Only works with the internal webserver.

profile
    show 10 most time consuming function in Isso after each request. Do
    not use in production.

.. _configure-smtp:

SMTP
----

Isso can notify you on new comments via SMTP. In the email notification, you
also can moderate (=activate or delete) comments. Don't forget to configure
``notify = smtp`` in the general section.

.. code-block:: ini

    [smtp]
    username =
    password =
    host = localhost
    port = 587
    security = starttls
    to =
    from =
    timeout = 10

username
    self-explanatory, optional

password
    self-explanatory (yes, plain text, create a dedicated account for
    notifications), optional.

host
    SMTP server

port
    SMTP port

security
    use a secure connection to the server, possible values: *none*, *starttls*
    or *ssl*. Note, that there is no easy way for Python 2.7 and 3.3 to
    implement certification validation and thus the connection is vulnerable to
    Man-in-the-Middle attacks. You should definitely use a dedicated SMTP
    account for Isso in that case.

.. _pac-smtp-to:

to
    recipient address, e.g. your email address

.. _pac-smtp-from:

from
    sender address, e.g. `"Foo Bar" <isso@example.tld>`

timeout
    specify a timeout in seconds for blocking operations like the
    connection attempt.

.. _configure-wechat:

Wechat
------

Isso可以在有新评论时通过微信通知您（依靠 `Server酱 <http://sc.ftqq.com>`_ 的
服务，英文名「ServerChan」，是一款「程序员」和「服务器」之间的通信软件）。
在微信通知中，您还可以审核（=激活或删除）评论。不要忘记在服务端INI文件
的 ``general`` 中配置 ``notify = wechat`` 。

.. note::

    这不是原Isso的功能，而是 `staugur/isso-cn <https://github.com/staugur/isso-cn>`_ 专门为国内用户新增的功能。

    所以需要安装新的Isso-cn，参考 :ref:`安装一节 <install-from-pypi>` ，从
    源码安装大致步骤如下：

    .. code-block:: bash

        # git clone https://github.com/staugur/isso-cn.git && cd isso-cn
        # npm install -g node-sass requirejs bower jade # or `yarn global add`
        # make init js
        # pip install .

服务端INI配置文件示例：

.. code-block:: ini

    [wechat]
    sckey = Server酱发送消息的SCKEY
    takey = Server酱TalkAdmin服务提供的WebHook回调地址的Key

.. _pac-sckey:

sckey
    使用Server酱发送消息的基本服务，您需要有一个密钥，即SCKEY。申请方法为：
        1. 打开：使用浏览器打开 `Server酱官网 <http://sc.ftqq.com>`_
        2. 登入：使用GitHub登入，在「发送消息」页面，就能看到您的 **SCKEY**
        3. 绑定：在「微信推送」页面，扫码关注公众号「方糖」的同时即可完成绑定。后面新消息就会推送到此公众号，当然只有您自己才能收到。

.. _pac-takey:

takey
    同样是由Server酱提供的另一款服务：TalkAdmin，它提供两个类型的命令，其
    文档是（大概看一眼有所了解）：http://sc.ftqq.com/5.version，在这里用的
    是下行命令。

    在Isso中，您需要在TalkAdmin页面添加命令，如图示：
        |talkadmin_new|

    - 交互界面模板的HTML代码是：
        .. code-block:: html

            <a href="{{$TA_activate}}" class="btn btn-primary font-white">通过 </a> &nbsp; | &nbsp;
            <a href="{{$TA_delete}}" class="btn btn-danger font-white">拒绝</a> &nbsp; | &nbsp;
            <a href="{{$TA_view}}" class="btn btn-info font-white">查看</a>

        参照此代码一般不用更改，代码中以 `TA_` 开头的变量绝对不要更改，
        其他样式参考官方文档编写。

    - 交互界面自定义CSS，可根据模板中代码调整样式，如：
        .. code-block:: css

            a.font-white {color:white!important}

    - 命令正则、WebHook地址不需要填。

    保存后，Server酱会自动生成WebHook地址，类似于 ``http://sc.ftqq.com/webhook/xxx``，
    这个末尾的xxx，就是Isso需要的 **takey** ！

.. note::

    以上两个key不需要同时提供！

    - 当 :ref:`审核功能 <configure-moderation>` 开启时，Isso会使用TalkAdmin服务，此时需要takey。
        有新评论时，调用Server酱向微信公众号「方糖」推送消息，绑定的微信收
        到消息，其内容包含评论页面标题、详细内容、IP等，另外还有三个按钮，
        分别是通过（激活）、拒绝（删除）、查看，用来审核新评论。

    - 当 :ref:`审核功能 <configure-moderation>` 未开启时，Isso仅使用Server酱发送消息，此时需要sckey。
        有新评论时，Server酱向微信公众号推送消息，内容与邮件提醒的类似。

.. |talkadmin_new| image:: /_static/talkadmin.png

.. _pac-guard:

Guard
-----

Enable basic spam protection features, e.g. rate-limit per IP address (``/24``
for IPv4, ``/48`` for IPv6).

.. code-block:: ini

    [guard]
    enabled = true
    ratelimit = 2
    direct-reply = 3
    reply-to-self = false
    require-author = false
    require-email = false

enabled
    enable guard, recommended in production. Not useful for debugging
    purposes.

.. _pac-ratelimit:

ratelimit
    limit to N new comments per minute.

.. _pac-direct-reply:

direct-reply
    how many comments directly to the thread (prevent a simple
    `while true; do curl ...; done`.

.. _pac-reply-to-self:

reply-to-self
    allow commenters to reply to their own comments when they could still edit
    the comment. After the editing timeframe is gone, commenters can reply to
    their own comments anyways.

    Do not forget to configure the `client <client>`_ accordingly

.. _pac-require-author:

require-author
    force commenters to enter a value into the author field. No validation is
    performed on the provided value.

    Do not forget to configure the `client <client>`_ accordingly.

.. _pac-require-email:

require-email
    force commenters to enter a value into the email field. No validation is
    performed on the provided value.

    Do not forget to configure the `client <client>`_ accordingly.

.. _pac-markup:

Markup
------

Customize markup and sanitized HTML. Currently, only Markdown (via Misaka) is
supported, but new languages are relatively easy to add.

.. code-block:: ini

    [markup]
    options = strikethrough, superscript, autolink
    allowed-elements =
    allowed-attributes =

options
    `Misaka-specific Markdown extensions <http://misaka.61924.nl/#api>`_, all
    flags starting with `EXT_` can be used there, separated by comma.

.. _pac-allowed-elements:

allowed-elements
    Additional HTML tags to allow in the generated output, comma-separated. By
    default, only *a*, *blockquote*, *br*, *code*, *del*, *em*, *h1*, *h2*,
    *h3*, *h4*, *h5*, *h6*, *hr*, *ins*, *li*, *ol*, *p*, *pre*, *strong*,
    *table*, *tbody*, *td*, *th*, *thead* and *ul* are allowed.

.. _pac-allowed-attributes:

allowed-attributes
    Additional HTML attributes (independent from elements) to allow in the
    generated output, comma-separated. By default, only *align* and *href* are
    allowed.

To allow images in comments, you just need to add ``allowed-elements = img`` and
``allowed-attributes = src``.

Hash
----

Customize used hash functions to hide the actual email addresses from
commenters but still be able to generate an identicon.

.. code-block:: ini

    [hash]
    salt = Eech7co8Ohloopo9Ol6baimi
    algorithm = pbkdf2

salt
    A salt is used to protect against rainbow tables. Isso does not make use of
    pepper (yet). The default value has been in use since the release of Isso
    and generates the same identicons for same addresses across installations.

algorithm
    Hash algorithm to use -- either from Python's `hashlib` or PBKDF2 (a
    computational expensive hash function).

    The actual identifier for PBKDF2 is `pbkdf2:1000:6:sha1`, which means 1000
    iterations, 6 bytes to generate and SHA1 as pseudo-random family used for
    key strengthening.
    Arguments have to be in that order, but can be reduced to `pbkdf2:4096`
    for example to override the iterations only.

.. _configure-rss:

RSS
---

Isso can provide an Atom feed for each comment thread. Users can use
them to subscribe to comments and be notified of changes. Atom feeds
are enabled as soon as there is a base URL defined in this section.

.. code-block:: ini

    [rss]
    base =
    limit = 100

base
    base URL to use to build complete URI to pages (by appending the URI from Isso)

limit
    number of most recent comments to return for a thread

.. _pac-admin:

Admin
-----

Isso has an optional web administration interface that can be used to moderate
comments. The interface is available under ``/admin`` on your isso URL.

.. code-block:: ini

    [admin]
    enabled = true
    password = secret

enabled
    whether to enable the admin interface

.. _pac-password:

password
    the plain text password to use for logging into the administration interface

Appendum
--------

.. _appendum-timedelta:

Timedelta
    A human-friendly representation of a time range: `1m` equals to 60
    seconds. This works for years (y), weeks (w), days (d) and seconds (s),
    e.g. `30s` equals 30 to seconds.

    You can add different types: `1m30s` equals to 90 seconds, `3h45m12s`
    equals to 3 hours, 45 minutes and 12 seconds (12512 seconds).

Environment variables
---------------------

.. _environment-variables:

Isso also support configuration through some environment variables:

ISSO_CORS_ORIGIN
    By default, `isso` will use the `Host` or else the `Referrer` HTTP header
    of the request to defines a CORS `Access-Control-Allow-Origin` HTTP header
    in the response.
    This environent variable allows you to define a broader fixed value,
    in order for example to share a single Isso instance among serveral of your
    subdomains : `ISSO_CORS_ORIGIN=*.example.test`
