---
title: "CS144 Lab 0"
published: 2023-09-10
description: 'C++ 不会捏😭'
image: ''
tags: [CS144, Computer Network]
category: 'CSDIY'
draft: false 
lang: ''
---

## Set up GNU/Linux on your computer

照着 [check0](https://cs144.github.io/assignments/check0.pdf) 上说的做就好了。

一开始用的是 Mac 装 UTM ，使用官方镜像，再设共享文件夹，好不容易搞定这一步之后 cmake 啥的各种报错，遂放弃。

只好掏出我的暗影精灵 8 ，安装 Virtual Box ，使用官方镜像，设置共享文件夹，注意这里共享文件夹每次开机之后要输入 `sudo mount -t vboxsf CS144 <YourSharedFolderPath>` 才能将你取名 CS144 的文件夹挂载到 `<YourSharedFolderPath>` ，好好好，这下环境终于没问题了。

## Networking by hand

照着做，除了华科的 Email 每次都给我关闭 connection 同时不让你用 smtp 发送邮件。

## Listening and connecting

还是照着做。

## Writing a network program using an OS stream socket

### Let\'s get started——fetching and building the starter code

第一步使用官方镜像就不会有问题。

### Modern C++: mostly safe but still fast and low-level

C++ 不会捏，只会基础 C with STL 呜呜

### Reading the Minnow support code

看看头文件里定义了哪些函数，各自有什么功能和 API 即可。

### Writing `webget`

网上查查 socket 编程大概是个什么流程，再根据 check0 文档和头文件的内容写就好了。

```cpp
void get_URL( const string& host, const string& path ) {
  TCPSocket socket {};
  socket.connect( Address(host, "http") );
  string message = "GET " + path + " HTTP/1.1\r\n";
  message += "Host: " + host + "\r\n";
  message += "Connection: close\r\n\r\n";
  socket.write(message);
  string rec_message;
  while (!socket.eof()) {
    socket.read(rec_message);
    cout << rec_message;
  }
  socket.close();

  cerr << "Function called: get_URL(" << host << ", " << path << ")\n";
  cerr << "Warning: get_URL() has not been implemented yet.\n";
}
```

### An in-memory reliable byte stream

这一步一开始看到差点没把我愁坏了......一度考虑是否需要先去补补 C++ 的语言基础，直到我看到了 [CS144 Lab0 笔记：环境配置和热身](https://zhangjk98.xyz/cs144-note/) ，感觉好像还好？

于是先写了个两个 deque ，一个是 string 另一个是 string_view ，结果倒在了 Test 8 ，也不知道到底错哪了，而且确实不会 string_view ，于是去掉了 string_view ，之后貌似 Test 4 又寄了？

看了一下，好像是爆栈了还是 string 存不下了来着（忘了 qwq），只好又去掉了 deque 使用纯 string ，终于才过掉了。

byte_stream.hh :

```cpp
class ByteStream
{
protected:
  uint64_t capacity_;
  // Please add any additional state to the ByteStream here, and not to the Writer and Reader interfaces.
  std::string buffer {};
  uint64_t buffered_bytes = 0;
  uint64_t pushed_bytes = 0;
  uint64_t poped_bytes = 0;
  bool _is_closed = 0;
  bool _has_error = 0;

public:
  explicit ByteStream( uint64_t capacity );

  // Helper functions (provided) to access the ByteStream's Reader and Writer interfaces
  Reader& reader();
  const Reader& reader() const;
  Writer& writer();
  const Writer& writer() const;
};
```

byte_stream.cc :

```cpp
void Writer::push( string data )
{
  // Your code here.
  (void)data;
  uint64_t data_length = data.length();
  if (is_closed() || data.empty()) return;
  if (available_capacity() < data_length) {
    data_length = available_capacity();
  }
  buffered_bytes += data_length;
  pushed_bytes += data_length;
  buffer += data.substr(0, data_length);
}

void Writer::close()
{
  // Your code here.
  _is_closed = 1;
}

void Writer::set_error()
{
  // Your code here.
  _has_error = 1;
}

bool Writer::is_closed() const
{
  // Your code here.
  return _is_closed;
}

uint64_t Writer::available_capacity() const
{
  // Your code here.
  return capacity_ - buffered_bytes;
}

uint64_t Writer::bytes_pushed() const
{
  // Your code here.
  return pushed_bytes;
}

string_view Reader::peek() const
{
  // Your code here.
  return buffer;
}

bool Reader::is_finished() const
{
  // Your code here.
  return _is_closed && buffered_bytes == 0;
}

bool Reader::has_error() const
{
  // Your code here.
  return _has_error;
}

void Reader::pop( uint64_t len )
{
  // Your code here.
  (void)len;
  if (len == 0) return;
  len = min(len, buffer.length());
  buffer.erase(0, len);
  buffered_bytes -= len;
  poped_bytes += len;
}

uint64_t Reader::bytes_buffered() const
{
  // Your code here.
  return buffered_bytes;
}

uint64_t Reader::bytes_popped() const
{
  // Your code here.
  return poped_bytes;
}
```

## References

- [CS144 writeup - 计算机网络 Lab0](https://www.expoli.tech/articles/2023/06/12/CS144-writeup-Lab-Checkpoint-0:-networking-warmup#78b1c827bf904d43b68ecce238b2e31f)
- [CS144 2023 完全指南](https://zhuanlan.zhihu.com/p/630739394)
- [CS144 Lab0 笔记：环境配置和热身](https://zhangjk98.xyz/cs144-note/)