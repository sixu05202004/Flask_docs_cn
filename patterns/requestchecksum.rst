请求内容校验
=========================

各种代码片断中可以消耗请求数据和处理它。比如JSON数据已经阅读并处理请求对象，表单数据结束了，但通过不同的代码路径。这似乎是不方便的，当你想要传入的请求数据来计算校验和。这是必要的，有时一些API。

幸运的是，然而，这是非常简单的更改，包装的输入流。

下面的例子计算SHA1校验传入的数据，因为它被读取，并将其存储在WSGI环境::

    import hashlib

    class ChecksumCalcStream(object):

        def __init__(self, stream):
            self._stream = stream
            self._hash = hashlib.sha1()

        def read(self, bytes):
            rv = self._stream.read(bytes)
            self._hash.update(rv)
            return rv

        def readline(self, size_hint):
            rv = self._stream.readline(size_hint)
            self._hash.update(rv)
            return rv

    def generate_checksum(request):
        env = request.environ
        stream = ChecksumCalcStream(env['wsgi.input'])
        env['wsgi.input'] = stream
        return stream._hash

要使用此功能，所有你需要做的是挂钩的计算请求之前开始消耗数据流。
（例如：小心访问的 ``request.form`` 或任何这种性质的。 例如 ``before_request_handlers`` 应小心，不要访问）。

使用示例::

    @app.route('/special-api', methods=['POST'])
    def special_api():
        hash = generate_checksum(request)
        # Accessing this parses the input stream
        files = request.files
        # At this point the hash is fully constructed.
        checksum = hash.hexdigest()
        return 'Hash was: %s' % checksum
