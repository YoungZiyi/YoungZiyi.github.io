---
title: PEP333——Python Web服务器网关接口
date: 2017-8-24 22:32:11 +0800
categories: PEP
tags: 翻译
---

# PEP 333——Python Web服务器网关接口 v1.0

Original: [PEP 333——Python Web Server Gateway Interface v1.0](https://www.python.org/dev/peps/pep-0333/#abstract)  

> PEP:	333  
> Title:	Python Web Server Gateway Interface v1.0  
> Author:	Phillip J. Eby <pje at telecommunity.com>  
> Discussions-To:	Python Web-SIG <web-sig at python.org>  
> Status:	Final  
> Type:	Informational  
> Created:	07-Dec-2003  
> Post-History:	07-Dec-2003, 08-Aug-2004, 20-Aug-2004, 27-Aug-2004, 27-Sep-2010  
> Superseded-By:	3333  

## 前言
注意：此份规范的更新版本支持python3.x、包含社区勘误表、附录表，以及说明，如果这是你想要的，那么去看[PEP 3333](https://www.python.org/dev/peps/pep-3333)。

## 摘要
这篇文档介绍了在服务器和python web应用程序（或框架）之间建立一套标准接口，使python web应用程序可以在多种web服务器之间运行，提高可移植性。

## 原理及目标
目前python拥有大量web应用框架，
例如Zope，Quixote，Webware，SkunkWeb，PSO，以及Twisted Web，
等等[1](http://www.python.org/cgi-bin/moinmoin/WebProgramming)，
数量众多的选择对新手来说是个头疼的问题，
因为，他们对web框架的选择将会限制所能使用的web服务器，反之亦然。  

相比之下，尽管java也有大量web应用框架，
但是java servlet API使得任何使用java框架写的应用程序都可以在任何支持java sevlet的web服务器上运行。

python也需要那样一套可用的广泛传播的API，
无论服务器是用python语言写的（Medusa），
或者把python封装进去（mod_python），
或者通过网关协议调用python解释器来执行程序（CGI,FastCGI,etc）都将使得对框架的选择脱离web服务器的限制，
让用户自由选择最适合他们的组合，
而且还能使框架和web服务器的开发者们专注于做好他们擅长的工作。

因此，这份PEP提议在web服务器和web应用程序或框架之间建立一套简单而通用的接口：
Python Web服务器网关接口（WSGI）。

但是仅仅起草一份WSGI规范并不能解决python web服务器和框架现有的状况，
服务器、框架作者和维护人员必须真正的实现WSGI才会有效果。

然而，由于还没有任何服务器或者框架支持WSGI，
最初实现对WSGI的支持对作者来说并没有立即产生效果。
因此，WSGI规范必须简洁，
至少能使作者实现接口的成本比较低。

因此，服务器和框架两端的接口实现起来都比较简单，这对WSGI接口的实用性都非常重要，
对任何设计决策来说都是基本原则。

注意，然而，框架实现对WSGI的支持并不像使用WSGI接口那般简单。
WSGI提供一个完全没有多余功能的接口给框架作者，
因为多余的类似响应对象和cookie的处理将会阻碍框架中已实现的相同功能。
再一次，WSGI的目标是使服务器和应用或框架之间交互更简单，而不是创造一个新的框架。

还要注意的是，这个目标阻止WSGI需要任何已部署python版本中不存在的东西。
因此，这份规范并不提议或者要求使用新标准库，
另外WSGI不需要任何高于python2.2.2版本的特性。
（对未来python版本中，最好是在标准库提供的web服务器中实现对接口的支持。）

另外，对现存以及未来的框架和服务器实现起来都应该简单，
它还要能够简单创建请求预处理器和响应后处理器（？），
其他基于WSGI的“中间件”对他们的服务器来说像一个应用，
对他们的应用来说却像一个服务器。

如果中间件能够既简单又强健，
而且WSGI被服务器和框架广泛支持，
那么它将允许一个全新的python web应用框架：
一个由低耦合WSGI中间件组件组成的框架。
甚至框架作者可能选择重构他们框架中的服务以提供上述架构，
使用WSGI更像使用一个库，
而不是一个庞大的框架。
这将使得应用开发者能够选择最佳组件以应对特殊的应用场景，
而不是不得不接受同一个框架中不好的一面。

当然，要实现上述愿景，
还有一条很长的路要走。
在此期间，有足够的短期目标使WSGI能够在任何框架和服务器上使用。

最后，需要提到的是，目前的WSGI版本并没有规定任何特殊的结构来使用web服务器或网关部署一个应用。
目前，这是由服务器或网关定义的。
当足够多的服务器和框架实现了WSGI，可以提供多种不同的部署需求，
那么需要创建另一份PEP，为WSGI服务器和应用框架描述一个部署标准。


## 规范概览

WSGI接口有两端：服务器或网关端，应用或框架端。
应用端提供一个可调用对象给服务器端调用。
如何提供这个可调用对象的细节则由服务器或网关决定。
假设一些服务器或网关会要求应用提供一个部署脚本，用来提供给服务器或网关创建一个实例，
来提供一个应用程序对象。
其他的服务器或网关可能使用配置文件或者其他结构来指定从哪里引入（或者说获得）应用程序对象。

另外为了使通信双方更加纯粹，
可以使用中间件来同时实现WSGI的两端。
那么他对服务器来说是一个应用，
对应用过来说则是扮演服务器的角色，
还能用来提供扩展APIs，内容转换，导航，以及其他有用的功能。

贯穿这篇规范说明，
我们会使用“可调用对象”来表示一个“函数”，“方法”，“类”或者一个有__call__方法的实例。
至于这个“可调用搞对象”具体是什么，这取决于服务器，网关，或者应用根据需要选择合适的技术来实现可调用对象。

### 应用端/框架端
应用对象就是一个简单的接受两个参数的可调用对象，
术语“对象”并不是特指一个真正的对象实例：
一个函数，方法，类或者实现了__call__方法的实例都可以作为一个应用对象。
应用对象必须要能够调用多次，
几乎所有的服务器或网关（除了CGI）都会有重复的请求。

>（注意：
> 虽然我们称它为应用对象，
> 但是应用开发者不应该把WSGI当作一个web programming API！
> 应用开发者们还是应该继续使用现有的，高级框架服务来开发他们的应用。
> WSGI是框架和服务器开发者们的工具，而且不打算直接支持应用开发者。）

这里有两个应用对象的例子；一个是函数，另一个是类：
``` python
def simple_app(environ, start_response):
    """最简单的应用对象"""
    status = '200 OK'
    response_headers = [('Content-type', 'text/plain')]
    start_response(status, response_headers)
    return ['Hello world!\n']

class AppClass:
    """实现和上面例子同样功能的应用对象，但是使用类来实现

    （注意：‘AppClass’是一个应用，通过调用它返回一个‘AppClass’的实例，
    一个通过可迭代返回值的应用对象。）

    如果我们想使用‘AppClass’的实例作为应用对象，那就应该实现__call__
    方法，那样就能用过调用实例来执行应用，而且需要创建一个实例供服务器
    或网关使用。
    """

    def __init__(self, environ, start_response):
        self.environ = environ
        self.start = start_response

    def __iter__(self):
        status = '200 OK'
        response_headers = [('Content-type', 'text/plain')]
        self.start(status, response_headers)
        yield "Hello world!\n"
```

### 服务器端/网关端

HTTP客户端每次发送一个请求，服务器或网关都会调用应用对象一次，
为了好理解，这里有一个简单的CGI网关，
通过一个接受应用对象的函数实现。
注意这个简单的例子只能处理有限的错误状况，
因为一个未捕捉的异常默认将会通过sys.stderr输出，并且由服务器记录日志。
``` python
import os, sys

def run_with_cgi(application):

    environ = dict(os.environ.items())
    environ['wsgi.input']        = sys.stdin
    environ['wsgi.errors']       = sys.stderr
    environ['wsgi.version']      = (1, 0)
    environ['wsgi.multithread']  = False
    environ['wsgi.multiprocess'] = True
    environ['wsgi.run_once']     = True

    if environ.get('HTTPS', 'off') in ('on', '1'):
        environ['wsgi.url_scheme'] = 'https'
    else:
        environ['wsgi.url_scheme'] = 'http'

    headers_set = []
    headers_sent = []

    def write(data):
        if not headers_set:
             raise AssertionError("write() before start_response()")

        elif not headers_sent:
             # Before the first output, send the stored headers
             status, response_headers = headers_sent[:] = headers_set
             sys.stdout.write('Status: %s\r\n' % status)
             for header in response_headers:
                 sys.stdout.write('%s: %s\r\n' % header)
             sys.stdout.write('\r\n')

        sys.stdout.write(data)
        sys.stdout.flush()

    def start_response(status, response_headers, exc_info=None):
        if exc_info:
            try:
                if headers_sent:
                    # Re-raise original exception if headers sent
                    raise exc_info[0], exc_info[1], exc_info[2]
            finally:
                exc_info = None     # avoid dangling circular ref
        elif headers_set:
            raise AssertionError("Headers already set!")

        headers_set[:] = [status, response_headers]
        return write

    result = application(environ, start_response)
    try:
        for data in result:
            if data:    # don't send headers until body appears
                write(data)
        if not headers_sent:
            write('')   # send headers now if body was empty
    finally:
        if hasattr(result, 'close'):
            result.close()
```

### 中间件：同时扮演两端角色的组件

“中间件”面对应用的时候扮演服务器的角色，而面对服务器的时候扮演应用的角色，
它提供以下功能：

+ 在相应的重写环境之后，根据目标URL把请求路由到不同的应用对象。
+ 允许多个应用或者框架并行的运行在同一个进程中。
+ 通过转发请求以及网络响应来实现负载平衡和远程处理。
+ 执行内容预处理，例如应用XSL样式表。

总之，中间件的存在是使“服务器/网关”端和“应用/框架”端之间透明，不需要任何特殊支持使两端能够交互。
想把中间件合并到应用的用户只需提供中间件组件给服务器，
中间件就好像是一个应用，配置好中间件使之去调用应用，
中间件又好像是一个服务器，代替服务器去调用应用。
当然，中间件封装的“应用”可能是另一个中间件封装的另一个应用，
如此循环，创建了所谓的“中间件堆栈”。

在极大程度上，中间件必须符合WSGI规范服务端和应用端的限制和要求。
在某些例子，对中间件的约束比单纯对服务器或应用还要严格，
这些要点将在规范中注明。

下面是一个中间件例子（非正式），它的作用是把文本转换成pig Latin式的儿童黑话（...?），
代码保存在Joe Strout的piglatin.py文件中。
（注意：一个健壮的中间件可能会使用更粗鲁的方式检查内容类型，
以及内容的编码格式。
另外，这个例子忽略了一个单词可能跨越块边界。）

``` python
from piglatin import piglatin

class LatinIter:

    """Transform iterated output to piglatin, if it's okay to do so

    Note that the "okayness" can change until the application yields
    its first non-empty string, so 'transform_ok' has to be a mutable
    truth value.
    """

    def __init__(self, result, transform_ok):
        if hasattr(result, 'close'):
            self.close = result.close
        self._next = iter(result).next
        self.transform_ok = transform_ok

    def __iter__(self):
        return self

    def next(self):
        if self.transform_ok:
            return piglatin(self._next())
        else:
            return self._next()

class Latinator:

    # by default, don't transform output
    transform = False

    def __init__(self, application):
        self.application = application

    def __call__(self, environ, start_response):

        transform_ok = []

        def start_latin(status, response_headers, exc_info=None):

            # Reset ok flag, in case this is a repeat call
            del transform_ok[:]

            for name, value in response_headers:
                if name.lower() == 'content-type' and value == 'text/plain':
                    transform_ok.append(True)
                    # Strip content-length if present, else it'll be wrong
                    response_headers = [(name, value)
                        for name, value in response_headers
                            if name.lower() != 'content-length'
                    ]
                    break

            write = start_response(status, response_headers, exc_info)

            if transform_ok:
                def write_latin(data):
                    write(piglatin(data))
                return write_latin
            else:
                return write

        return LatinIter(self.application(environ, start_latin), transform_ok)


# Run foo_app under a Latinator's control, using the example CGI gateway
from foo_app import foo_app
run_with_cgi(Latinator(foo_app))

```


## 规范详述

一个应用对象必须接受两个位置参数。
为了好理解，
我们分别给这两个参数取名为environ和start_response，
但是这也不是必须的。
服务器或网关必须用这两个位置参数去调用一个应用（例如，通过调用`result = application(environ, start_response)`）

environ参数是一个字典对象，
包含类似CGI环境参数。
这个对象必须是一个python内置的字典对象（不是一个子类，自定义字典类或者其他模拟字典类），
而且应用能够随意修改字典的内容。
字典还必须包含必要的WSGI变量（下节将会介绍），
以及包含服务器端扩展变量，将根据下面描述的来命名。

start_response参数是一个可调用对象，它接受两个必选参数和一个可选参数。
为了更好的理解，
我们分别命名这些参数status，response_headers，和exc_info，
但是他们同样不是必须叫这个名，
应用必须调用start_response对象，并且使用位置参数（例如，start_response(status, response_headers)）.

status参数是一个表状态的字符串，类似“999 Message here”，
response_headers是一个包含http响应头的元祖。
可选的exc_info参数在下节start_reponse()可调用对象和错误处理中介绍。
它仅在当应用程序出错或者打算在浏览器上显示错误信息的时候使用。

`start_response()`必须返回一个`write`（主体数据）可调用对象，它接受一个位置参数：
HTTP响应主体的一部分字符串。
（注意：`write()`可调用对象仅用于某些现有框架必须的输出APIs；
如果可以避免，新的应用或框架就不应该适用这个对象。
参阅Buffering and Stream章节获取更多信息。）

当服务器调用时，
应用对象必须返回一个可迭代对象，返回0或更多字符串。
这可以通过很多方式来实现，
例如返回字符串列表
或者通过一个生成器函数实现应用程序对象，
或者用一个可迭代类实例来实现应用程序对象。
无论它时怎么实现的，
应用程序对象必须返回一个可迭代对象，返回0或更多字符串。

服务器或网关必须以非缓存的方式返回生成的字符串到客户机，
在另一个请求前，完成每一个字符串的返回。
（换句话说，应用必须自己做好缓存,
参阅Buffering and Stream章节获取更多有关应用程序输出处理的信息。）

服务器或网关按二进制字节序列来处理来处理生成的字符串：
尤其是，它应该确保行尾结束符不能被修改。
应用对象响应的数据必须必须适合在客户端展现。
（服务器或网关可能应用HTTP传输编码。或者其他实现HTTP功能的传输方式，例如字节范围传输。参阅下面的Other HTTP Features章节获取更多信息。）

如果能够调用`len(iterable)`，
服务器必须能够信任结果的准确度。
如果可迭代对象是通过应用提供的`__len__()`方法返回的，
则必须返回一个准确的结果。
（参阅Handling the Content-Length Header章节获取更多信息。）

如果应用返回的可迭代对象有一个`close()`方法，
服务器或网关必须调用那个方法以完成当前请求，
无论请求是否已经正常完成，或者由于错误提前终止
（这是用来支持应用程序的资源发布）。
这个协议是补充PEP 325的生成器支持章节，
以及其他常见有`close()`方法的可迭代对象。

（注意：应用必须在可迭代开始生成主题字符串之前调用`start_response()`，使得服务器能够在主体内容之前发送headers。）
然而，这个调用可以通过可迭代对象的首次迭代来调用实现，
所以，服务器必须不假设应用首先调用`start_response()`，然后在开始迭代。）

最后，服务器和网关必须不直接使用应用返回的可迭代对象的其他属性。
除非它在服务器或网关中是一个特别的实例，有自有的特别属性，
例如wsgi.file_wrapper返回“file wrapper”
（参阅Optional Platform-Specific File Handling）。
通常情况下，只有这里指定的属性，
或者通过例如PEP 234中的迭代APIs。

### 变量`environ`

environ字典需要包含CGI环境变量，
定义在Common Gateway Interface规范[2](http://ken.coar.org/cgi/draft-coar-cgi-v11-03.txt)中。
下面的变量必须提供，
除非他们的值是空字符串，
则可能可以省略，除了额外注明的变量。

> + REQUEST_METHOD  
> The HTTP request method, such as "GET" or "POST". This cannot ever be an empty string, and so is always required.  
>  
> + SCRIPT_NAME  
> The initial portion of the request URL's "path" that corresponds to the application object, so that the application knows its virtual "location". This may be an empty string, if the application corresponds to the "root" of the server.  
>  
> + PATH_INFO  
> The remainder of the request URL's "path", designating the virtual "location" of the request's target within the application. This may be an empty string, if the request URL targets the application root and does not have a trailing slash.  
>  
> + QUERY_STRING  
> The portion of the request URL that follows the "?", if any. May be empty or absent.  
>  
> + CONTENT_TYPE  
> The contents of any Content-Type fields in the HTTP request. May be empty or absent.  
>  
> + CONTENT_LENGTH  
> The contents of any Content-Length fields in the HTTP request. May be empty or absent.  
>  
> + SERVER_NAME, SERVER_PORT  
> When combined with SCRIPT_NAME and PATH_INFO, these variables can be used to complete the URL. Note, however, that HTTP_HOST, if present, should be used in preference to SERVER_NAME for reconstructing the request URL. See the URL Reconstruction section below for more detail. SERVER_NAME and SERVER_PORT can never be empty strings, and so are always required.  
>  
> + SERVER_PROTOCOL  
> The version of the protocol the client used to send the request. Typically this will be something like "HTTP/1.0" or "HTTP/1.1" and may be used by the application to determine how to treat any HTTP request headers. (This variable should probably be called REQUEST_PROTOCOL, since it denotes the protocol used in the request, and is not necessarily the protocol that will be used in the server's response. However, for compatibility with CGI we have to keep the existing name.)  
>  
> + HTTP_ Variables  
> Variables corresponding to the client-supplied HTTP request headers (i.e., variables whose names begin with "HTTP_"). The presence or absence of these variables should correspond with the presence or absence of the appropriate HTTP header in the request.  

A server or gateway should attempt to provide as many other CGI variables as are applicable. In addition, if SSL is in use, the server or gateway should also provide as many of the Apache SSL environment variables [5] as are applicable, such as HTTPS=on and SSL_PROTOCOL. Note, however, that an application that uses any CGI variables other than the ones listed above are necessarily non-portable to web servers that do not support the relevant extensions. (For example, web servers that do not publish files will not be able to provide a meaningful DOCUMENT_ROOT or PATH_TRANSLATED.)

A WSGI-compliant server or gateway should document what variables it provides, along with their definitions as appropriate. Applications should check for the presence of any variables they require, and have a fallback plan in the event such a variable is absent.

Note: missing variables (such as REMOTE_USER when no authentication has occurred) should be left out of the environ dictionary. Also note that CGI-defined variables must be strings, if they are present at all. It is a violation of this specification for a CGI variable's value to be of any type other than str.

In addition to the CGI-defined variables, the environ dictionary may also contain arbitrary operating-system "environment variables", and must contain the following WSGI-defined variables:

| Variable | Value |
|----------|-------|
|wsgi.version|The tuple (1, 0), representing WSGI version 1.0.|
|wsgi.url_scheme|A string representing the "scheme" portion of the URL at which the application is being invoked. Normally, this will have the value "http" or "https", as appropriate.|
|wsgi.input|An input stream (file-like object) from which the HTTP request body can be read. (The server or gateway may perform reads on-demand as requested by the application, or it may pre- read the client's request body and buffer it in-memory or on disk, or use any other technique for providing such an input stream, according to its preference.)|
|wsgi.errors|An output stream (file-like object) to which error output can be written, for the purpose of recording program or other errors in a standardized and possibly centralized location. This should be a "text mode" stream; i.e., applications should use "\n" as a line ending, and assume that it will be converted to the correct line ending by the server/gateway.    <br><br>    For many servers, wsgi.errors will be the server's main error log. Alternatively, this may be sys.stderr, or a log file of some sort. The server's documentation should include an explanation of how to configure this or where to find the recorded output. A server or gateway may supply different error streams to different applications, if this is desired.|
|wsgi.multithread	|This value should evaluate true if the application object may be simultaneously invoked by another thread in the same process, and should evaluate false otherwise.|
|wsgi.multiprocess	|This value should evaluate true if an equivalent application object may be simultaneously invoked by another process, and should evaluate false otherwise.|
|wsgi.run_once	|This value should evaluate true if the server or gateway expects (but does not guarantee!) that the application will only be invoked this one time during the life of its containing process. Normally, this will only be true for a gateway based on CGI (or something similar).|

Finally, the environ dictionary may also contain server-defined variables. These variables should be named using only lower-case letters, numbers, dots, and underscores, and should be prefixed with a name that is unique to the defining server or gateway. For example, mod_python might define variables with names like mod_python.some_variable.

### Input and Error Streams


## 参考
[1] The Python Wiki "Web Programming" topic (http://www.python.org/cgi-bin/moinmoin/WebProgramming)
[2] The Common Gateway Interface Specification, v 1.1, 3rd Draft (http://ken.coar.org/cgi/draft-coar-cgi-v11-03.txt)
