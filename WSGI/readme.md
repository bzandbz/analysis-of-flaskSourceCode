## wsgi

全称： web service gateway interface

    可理解为是一个服务网关接口，介于发起请求的客户端和处理请求并返回响应的应用程序之间

### 出现wsgi的原因
    我们都知道，客服端与服务端通信，需要接受HTTP请求、解析HTTP请求、
    发送HTTP响应等，这都是相当底层的代码。而对我们大部分程序员来说，
    我们只需要关注业务逻辑的实现就可以了，我们不希望接触到TCP连接、
    HTTP原始请求和响应格式。

    因此，wsgi应运而生！

    由它来提供一个接口，
    让它去做上述的那些苦力活，做完了之后，把处理好的请求和响应格式等发送给
    我们的应用程序岂不美哉。

### 主流的wsgi

- uwsgi
- gunicorn

## wsgi的两部分


### 与客户端完成通信，处理数据
    接收客户端传递的数据，如请求路径，请求方法，请求数据等，打包成一个environ对象
    构建一个可调用的start_response方法

### 调用应用程序
    要求应用程序是可调用的，并且接收两个参数，分别是environ和start_response
    由应用程序返回响应
- environ 是一个字典，里面储存了HTTP request相关的所有内容，比如header、请求参数等等
- start_response是一个WSGI 服务器传递过来的函数，用于将response header，状态码传递给Server。
    

## wsgi工作流程
1. 初始化，接收两个参数：server_address（host+port）和 appplication(应用程序)
2. 创建一个socket对象，对server_address进行绑定监听。
3. 将传入的appplication赋予给初始化的server对象，等待对其调用。
4. socket对象开始循环接收客户端发送到server_address的请求
5. 接收到请求之后，建立链接，开始处理请求
6. 接收请求中的数据，包括请求方法，路径，http版本，请求头的一些信息参数，并打包成一个字典（environ对象）
7. 实现一个可调用函数（start_response）,该方法接收两个参数，分别是status,response_headers, 
    将由应用进行赋值。值得一提的是，在该方法中，会由wsgi默认一些header,如日期，wsgi版本号等。
    当应用传入response_header时，会与默认的header进行合并。
8. 调用application,传入environ和start_response参数，接收返回的结果（包括状态码，响应头，响应数据）
9. 对application返回的结果进行处理，形成真正的响应数据，返回给客户端。


