## io的定义

1. Block-IO:InputStream和OutputStream,Reader和Writer。属于同步阻塞模型，基于单向stream，性能低，按照传输方向可以分为InputStream和OutPutStream，父类Reader Writer。字符流。按照实现功能可以分为处理流和节点流。InputStreamWirter和InputStream。



### NIO常用类

Selector选择器

Channel通道

Buffer缓冲区

Pipe管道



### NIO模型

- select
  select模型只能支持1024
- poll
- epoll



## fileChannel和mmap