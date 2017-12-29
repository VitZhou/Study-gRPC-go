# Go gRPC Middleware

```go
import "github.com/grpc-ecosystem/go-grpc-middleware"
```

grpc_middleware是一个中间件包的集合： interceptors, helpers and tools.

## 中间件

gRPC是一个非常棒的RPC中间件，在Golang世界看到了很多应用。 但是，上游的gRPC代码库是相对裸露的。

这个包和大部分子包为gRPC提供了通用的中间件：用于retires的客户端拦截器，用于输入验证和auth的服务器端拦截器，用于链接所述拦截器的函数，元数据便利方法等等。

## Chaining

默认情况下，gRPC不允许在客户端和服务器端拥有多个拦截器。 grpc_middleware提供了方便的链接方法

将多个拦截器变成单个拦截器的简单方法。 这是一个服务器链接的例子：

```go
myServer := grpc.NewServer(
    grpc.StreamInterceptor(grpc_middleware.ChainStreamServer(loggingStream, monitoringStream, authStream)),
    grpc.UnaryInterceptor(grpc_middleware.ChainUnaryServer(loggingUnary, monitoringUnary, authUnary),
)
```

这些拦截器将从左到右执行：logging，monitoring和auth。

这里是一个客户端链接的例子：

```go
clientConn, err = grpc.Dial(
    address,
        grpc.WithUnaryInterceptor(grpc_middleware.ChainUnaryClient(monitoringClientUnary, retryUnary)),
        grpc.WithStreamInterceptor(grpc_middleware.ChainStreamClient(monitoringClientStream, retryStream)),
)
client = pb_testproto.NewTestServiceClient(clientConn)
resp, err := client.PingEmpty(s.ctx, &myservice.Request{Msg: "hello"})
```

这些拦截器将从左到右执行：monitoring然后重试逻辑。

## 自定义拦截器

实现你自己的拦截器是相当简单的：他们是一个interface。 但是有趣的一点是公共数据暴露给处理器（和其他中间件)，类似于HTTP中间件设计。 例如，您可能希望将来自auth拦截器的调用者的身份一直传递到处理函数。

例如，用于auth的客户端拦截器示例如下所示：

```go
func FakeAuthUnaryInterceptor(ctx context.Context, req interface{}, info *grpc.UnaryServerInfo, handler grpc.UnaryHandler) (interface{}, error) {
   newCtx := context.WithValue(ctx, "user_id", "john@example.com")
   return handler(newCtx, req)
}
```

不幸的是，流式RPC并不容易。 这些具有嵌入在grpc.ServerStream对象内的context.Context。 要通过上下文传递值，需要包装器（WrappedServerStream）。 例如：

```go
func FakeAuthStreamingInterceptor(srv interface{}, stream grpc.ServerStream, info *grpc.StreamServerInfo, handler grpc.StreamHandler) error {
   newStream := grpc_middleware.WrapServerStream(stream)
   newStream.WrappedContext = context.WithValue(ctx, "user_id", "john@example.com")
   return handler(srv, stream)
}
```

## 导入的包

- [golang.org/x/net/context]()
- [google.golang.org/grpc](https://godoc.org/google.golang.org/grpc)

### 包文件

- [chain.go](https://github.com/grpc-ecosystem/go-grpc-middleware/blob/master/chain.go)
- [doc.go](https://github.com/grpc-ecosystem/go-grpc-middleware/blob/master/doc.go)
- [wrappers.go](https://github.com/grpc-ecosystem/go-grpc-middleware/blob/master/wrappers.go)

### func [ChainStreamClient](https://github.com/grpc-ecosystem/go-grpc-middleware/blob/master/chain.go#L130)

```go
func ChainStreamClient(interceptors ...grpc.StreamClientInterceptor) grpc.StreamClientInterceptor
```

ChainStreamClient从许多拦截器链中创建一个单一的拦截器。

执行按照从左到右的顺序完成，包括Context的传递。 例如ChainStreamClient（一，二，三）将在三和二之前执行一。

### func [ ChainStreamServer](https://github.com/grpc-ecosystem/go-grpc-middleware/blob/master/chain.go#L56)

```go
func ChainStreamServer(interceptors ...grpc.StreamServerInterceptor) grpc.StreamServerInterceptor
```

ChainStreamServer从一系列拦截器中创建一个拦截器。

执行按照从左到右的顺序完成，包括上下文的传递。 例如ChainUnaryServer（一，两，三）将在三和二之前执行一。 如果要在拦截器之间传递上下文，请使用WrapServerStream。

### func [ChainUnaryClient](https://github.com/grpc-ecosystem/go-grpc-middleware/blob/master/chain.go#L93)

```go
func ChainUnaryClient(interceptors ...grpc.UnaryClientInterceptor) grpc.UnaryClientInterceptor
```

ChainUnaryClient从许多拦截器链中创建一个单一的拦截器。

执行按照从左到右的顺序完成，包括上下文的传递。 例如ChainUnaryClient（一，二，三）将在三和二之前执行一。

### func [ChainUnaryServer](https://github.com/grpc-ecosystem/go-grpc-middleware/blob/master/chain.go#L18)

```go
func ChainUnaryServer(interceptors ...grpc.UnaryServerInterceptor) grpc.UnaryServerInterceptor
```

ChainUnaryServer从一系列拦截器中创建一个拦截器。

执行按照从左到右的顺序完成，包括上下文的传递。 例如，ChainUnaryServer（一，二，三）将在三和二之前执行一，三个将看到一和二的上下文变化。

### func [WithStreamServerChain](https://github.com/grpc-ecosystem/go-grpc-middleware/blob/master/chain.go#L173)

```go
func WithStreamServerChain(interceptors ...grpc.StreamServerInterceptor) grpc.ServerOption
```

WithStreamServerChain是一个grpc.Server配置选项，可以接受多个流拦截器。 基本上是语法糖。

### fun [WithUnaryServerChain](https://github.com/grpc-ecosystem/go-grpc-middleware/blob/master/chain.go#L167)

```go
func WithUnaryServerChain(interceptors ...grpc.UnaryServerInterceptor) grpc.ServerOption
```

Chain从许多拦截器链中创建一个单一的拦截器。

WithUnaryServerChain是一个grpc.Server配置选项，接受多个unary拦截器。 基本上是语法糖

### func [WrappedServerStream](https://github.com/grpc-ecosystem/go-grpc-middleware/blob/master/wrappers.go#L12-L16)

```go
type WrappedServerStream struct {
    grpc.ServerStream
    // WrappedContext is the wrapper's own Context. You can assign it.
    WrappedContext context.Context
}
```

WrappedServerStream是grpc.ServerStream的一个简单的包装，允许修改上下文。

### func [WrapServerStream](https://github.com/grpc-ecosystem/go-grpc-middleware/blob/master/wrappers.go#L24)

```go
func WrapServerStream(stream grpc.ServerStream) *WrappedServerStream
```

WrapServerStream返回一个能够覆盖上下文的ServerStream。

### func (*WrappedServerStream)[Context](https://github.com/grpc-ecosystem/go-grpc-middleware/blob/master/wrappers.go#L19)

```go
func (w *WrappedServerStream) Context() context.Context
```

Context返回包装器的WrappedContext，覆盖嵌套的grpc.ServerStream.Context（）