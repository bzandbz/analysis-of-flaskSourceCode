## Werkzeug（flask框架的基础）

介绍Werkzeug之前需要强调一点：werkzeug不是一个web服务器，也不是一个web框架，
而是一个专门用来处理HTTP和WSGI的工具库。在设计之初的定位就是工具库。


### Werkzeug自带的wsgi服务器

在我们创建了一个flask app之后，我们在开发或者测试功能的时候，经常 `python app.py`来启动我们的flask程序。

app.py
```
from flask import Flask
app = Flask(__name__)

@app.route(‘/‘)
def index():
    return ‘Hello, world!’

if __name__ == ‘__main__’:
    app.run()

```

实际上执行的是app.run()方法。但是，此处的run方法是什么呢？实际上就是调用了werkzeug提供的run_simple方法。
```
def run(self, host=None, port=None, debug=None, load_dotenv=True, **options):
    ...
    ...
    ...

    from werkzeug.serving import run_simple

    try:
        run_simple(host, port, self, **options)
    finally:
        # reset the first request information if the development server
        # reset normally.  This makes it possible to restart the server
        # without reloader and that stuff from an interactive shell.
        self._got_first_request = False
```

这个run_simple方法的底层代码中，最终是调用了make_server方法，实现了一个srv对象，然后就可以进行监听了。
```

def make_server(
    host=None,
    port=None,
    app=None,
    threaded=False,
    processes=1,
    request_handler=None,
    passthrough_errors=False,
    ssl_context=None,
    fd=None,
):
    """Create a new server instance that is either threaded, or forks
    or just processes one request after another.
    """
    if threaded and processes > 1:
        raise ValueError("cannot have a multithreaded and multi process server.")
    elif threaded:
        return ThreadedWSGIServer(
            host, port, app, request_handler, passthrough_errors, ssl_context, fd=fd
        )
    elif processes > 1:
        return ForkingWSGIServer(
            host,
            port,
            app,
            processes,
            request_handler,
            passthrough_errors,
            ssl_context,
            fd=fd,
        )
    else:
        return BaseWSGIServer(
            host, port, app, request_handler, passthrough_errors, ssl_context, fd=fd
        )
```

注意：这个srv仅仅是werkzeug提供的仅满足基础wsgi功能的Server,是在不考虑安全性或性能的情况下开发的。
所以绝对不能用于生产环境。在生产环境中，应使用wsgi应用服务器（如uwsgi或gunicorn）来提高性能，
并通过真实的Web服务器（如nginx）代理它以提高性能和安全性。Web服务器擅长对请求/响应进行排队，
可以同时提供静态和其他内容，并且设计用于处理SSL。WSGI服务器擅长有效地协调应用程序中的多个请求。

### Werkzeug工具库
官方的介绍说是一个WSGI工具包，它可以作为一个Web框架的底层库， 因为它封装好了很多Web框架的东西，例如 Request，Response 等等。
