# day16-完善业务逻辑自定义

在day14中，我们使`Connection`类以`OnConnect`回调函数的方式初步支持了业务逻辑自定义，我们自定义的业务逻辑是从服务器端可读事件触发后开始进入，所以需要自己处理读取数据的逻辑。这显然不合理，怎样事件触发、读取数据、异常处理等流程应该是网络库提供的基本功能，用户只应当关注怎样处理业务即可，所以业务逻辑的进入点应该是服务器读取完客户端的所有数据之后。这是，客户端传来的请求在`Connection`类的读缓冲区里，我们只需要根据请求来分发、处理业务即可。

我们通过设置`OnMessage`回调函数来自定义自己的业务逻辑，在服务器完全接收到客户端的数据之后，该函数触发。以下是一个echo服务器的业务逻辑：

```cpp
server->OnMessage([](Connection *conn){
  std::cout << "Message from client " << conn->ReadBuffer() << std::endl;
  if(conn->GetState() == Connection::State::Connected){
    conn->Send(conn->ReadBuffer());
  }
});
```

在进入该函数前，服务器已经完成了接受客户端数据并保存在读缓冲区里，业务逻辑只需要将读缓冲区里的数据发送回即可。

这样的设计更加符合服务器的功能准则与设计准则，进一步简化了服务器编程、隐藏了更多细节，使用者只需要完全关注自己核心的业务逻辑。