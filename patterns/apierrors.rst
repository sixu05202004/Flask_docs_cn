实现 API 异常
===========================

在Flask上层实现 RESTful APIs 是非常常见的。开发者碰到的第一件事情就是内置的异常对于 APIs 
是表达性不足并且使用的内容格式 ``text/html`` 对于 API 使用者用处不大。

比 ``abort``  更好的解决方案就是实现自己的异常类型和安装的错误处理器，产生用户期待错误的格式。

简单的异常类
----------------------

其基本思路是引入一个新的异常，该异常可以采取一个适当的人类可读的消息，一个状态的错误代码和
一些可选的提供更多的错误上下文内容。

这是一个简单的例子::

    from flask import jsonify

    class InvalidUsage(Exception):
        status_code = 400

        def __init__(self, message, status_code=None, payload=None):
            Exception.__init__(self)
            self.message = message
            if status_code is not None:
                self.status_code = status_code
            self.payload = payload

        def to_dict(self):
            rv = dict(self.payload or ())
            rv['message'] = self.message
            return rv

视图现在可以引起一个带有错误信息的异常。另外一些附加的东西可以通过 `payload` 参数以字典形式提供。

注册一个错误处理器
----------------------------

在这个时候视图可以引起一个错误，但是它会立即导致内部服务器错误。原因是没有为这个错误类注册处理器。::

    @app.errorhandler(InvalidAPIUsage)
    def handle_invalid_usage(error):
        response = jsonify(error.to_dict())
        response.status_code = error.status_code
        return response

视图中的用法
--------------

这里是视图使用这个函数的方法::

    @app.route('/foo')
    def get_foo():
        raise InvalidUsage('This view is gone', status_code=410)
