# 使用gRPC拦截器进行Prometheus监控

[Prometheus](https://prometheus.io/)监控您的[gRPC Go](https://github.com/grpc/grpc-go)服务器和客户端。

[grpc-ecosystem/java-grpc-prometheus](https://github.com/grpc-ecosystem/java-grpc-prometheus)是[gRPC Java](https://github.com/grpc/grpc-java)（相同度量，相同语义）的姊妹实现。

## 拦截器

gRPC Go最近版本支持了拦截器，即在请求传递到用户的应用程序逻辑之前，由gRPC服务器执行中间逻辑。 这是实现常见模式的完美方式：auth，logging和... monitoring。

要使用链式拦截器，请参阅[go-grpc-middleware](https://github.com/mwitkow/go-grpc-middleware)。
### 使用

有两种类型的拦截器：客户端和服务器端。 这个源码包提供了两个监控拦截器。

#### 服务端

```go
import "github.com/grpc-ecosystem/go-grpc-prometheus"
...
    // 初始化你的grpc拦截器
    myServer := grpc.NewServer(
        grpc.StreamInterceptor(grpc_prometheus.StreamServerInterceptor),
        grpc.UnaryInterceptor(grpc_prometheus.UnaryServerInterceptor),
    )
    // 注册你的grpc服务实现
    myservice.RegisterMyServiceServer(s.server, &myServiceImpl{})
    // 在所有的拦截器注册之后，确保所有的prometheus指标都被初始化。
    grpc_prometheus.Register(myServer)
    // 注册prometheus metrics处理器
    http.Handle("/metrics", promhttp.Handler())
...
```

#### 客户端

```go
import "github.com/grpc-ecosystem/go-grpc-prometheus"
...
   clientConn, err = grpc.Dial(
       address,
		   grpc.WithUnaryInterceptor(UnaryClientInterceptor),
		   grpc.WithStreamInterceptor(StreamClientInterceptor)
   )
   client = pb_testproto.NewTestServiceClient(clientConn)
   resp, err := client.PingEmpty(s.ctx, &myservice.Request{Msg: "hello"})
...
```


## Metrics

### Labels

所有服务器端指标都以grpc_server作为Prometheus子系统名称开始。 所有客户端指标都以grpc_client开头。 他们都有mirror(镜像)的概念。 同样，所有方法都包含相同的丰富标签：

- grpc_service: gRPC服务名称，它是protobuf包和grpc_service部分名称的组合。 例如。 对于包mwitkow.testproto和服务TestService的标签将grpc_service =“mwitkow.testproto.TestService”
- grpc-method: 在gRPC服务上调用的方法的名称。 例如。grpc_method=“Ping”
- grpc-request: gRPC类型的请求。 区分这两者对于延迟测量尤其重要。
	- unary: 单请求，单响应RPC
	- client_stream: 是多请求，单响应RPC
	- server_stream: 单一请求，多响应RPC
	- bidi_stream: 是多请求，多响应的RPC

另外对于已完成的RPC，使用以下标签：

- grpc-code:  可读的gRPC状态码。 所有状态的列表都很长，但是这里有一些常见的状态：
	- OK: 意味着RPC服务是成功的
	- IllegalArgument：  RPC包含错误的值
	- Internal: 服务器端的错误没有透露给客户

### Counters

计数器和他们的最新文档是在[server_reporter.go](https://github.com/grpc-ecosystem/go-grpc-prometheus/blob/master/server_reporter.go)和[client_reporter.go](https://github.com/grpc-ecosystem/go-grpc-prometheus/blob/master/client_reporter.go)各自的Prometheus处理器（通常是 /metrics）。

为了本文的目的，我们只讨论grpc_server指标。 grpc_client包含的镜像概念。

为了简单起见，我们假设我们正在跟踪一个服务器端的RPC调用mwitkow.testproto.TestService.PingList()。 调用成功并返回流中的20条消息

首先，在服务器接收到调用之后，立即增加grpc_server_started_total并启动处理时钟（如果启用了直方图

```xml
grpc_server_started_total{grpc_method="PingList",grpc_service="mwitkow.testproto.TestService",grpc_type="server_stream"} 1
```

然后调用用户逻辑。 它从包含请求的客户端收到一条消息（这是一个server_stream）：

```xml
grpc_server_msg_received_total{grpc_method="PingList",grpc_service="mwitkow.testproto.TestService",grpc_type="server_stream"} 1
```

用户逻辑可能会返回一个错误，或将多条消息发送回客户端。 在这种情况下，发回的20个消息中的每一个都会增加一个计数器：

```xml
grpc_server_msg_sent_total{grpc_method="PingList",grpc_service="mwitkow.testproto.TestService",grpc_type="server_stream"} 20
```

调用完成后，状态（OK或其他gRPC状态码）和相关的调用标签递增grpc_server_handled_total计数器。

```xml
grpc_server_handled_total{grpc_code="OK",grpc_method="PingList",grpc_service="mwitkow.testproto.TestService",grpc_type="server_stream"} 1
```

### 直方图(Histograms)

[Promethues Histograms]()是测量PRC延迟分布的好方法,但是如果它具有很高的基数,就不是一种值得推荐的做法了,默认情况下，延迟监视metries是禁用的.如果要启用，请在服务器初始化代码中调用以下内容:

```go
grpc_prometheus.EnableHandlingTimeHistogram()
```

调用完成后，处理时间将记录在Prometheus直方图变量grpc_server_handling_seconds中。 它包含三个子度量标准：

- grpc_server_handling_seconds_count - 按状态和方法计算的所有已完成RPC
- grpc_server_handling_seconds_sum - 按状态和方法计算RPC的累积时间，用于计算平均处理时间
- grpc_server_handling_seconds_bucket - 包含在各个处理时间桶中按状态和方法计算的RPC计数。 Prometheus可以使用这些桶来估计SLA（[见这里](https://prometheus.io/docs/practices/histograms/)）

计数器值将如下所示：

```xml
grpc_server_handling_seconds_bucket{grpc_code="OK",grpc_method="PingList",grpc_service="mwitkow.testproto.TestService",grpc_type="server_stream",le="0.005"} 1
grpc_server_handling_seconds_bucket{grpc_code="OK",grpc_method="PingList",grpc_service="mwitkow.testproto.TestService",grpc_type="server_stream",le="0.01"} 1
grpc_server_handling_seconds_bucket{grpc_code="OK",grpc_method="PingList",grpc_service="mwitkow.testproto.TestService",grpc_type="server_stream",le="0.025"} 1
grpc_server_handling_seconds_bucket{grpc_code="OK",grpc_method="PingList",grpc_service="mwitkow.testproto.TestService",grpc_type="server_stream",le="0.05"} 1
grpc_server_handling_seconds_bucket{grpc_code="OK",grpc_method="PingList",grpc_service="mwitkow.testproto.TestService",grpc_type="server_stream",le="0.1"} 1
grpc_server_handling_seconds_bucket{grpc_code="OK",grpc_method="PingList",grpc_service="mwitkow.testproto.TestService",grpc_type="server_stream",le="0.25"} 1
grpc_server_handling_seconds_bucket{grpc_code="OK",grpc_method="PingList",grpc_service="mwitkow.testproto.TestService",grpc_type="server_stream",le="0.5"} 1
grpc_server_handling_seconds_bucket{grpc_code="OK",grpc_method="PingList",grpc_service="mwitkow.testproto.TestService",grpc_type="server_stream",le="1"} 1
grpc_server_handling_seconds_bucket{grpc_code="OK",grpc_method="PingList",grpc_service="mwitkow.testproto.TestService",grpc_type="server_stream",le="2.5"} 1
grpc_server_handling_seconds_bucket{grpc_code="OK",grpc_method="PingList",grpc_service="mwitkow.testproto.TestService",grpc_type="server_stream",le="5"} 1
grpc_server_handling_seconds_bucket{grpc_code="OK",grpc_method="PingList",grpc_service="mwitkow.testproto.TestService",grpc_type="server_stream",le="10"} 1
grpc_server_handling_seconds_bucket{grpc_code="OK",grpc_method="PingList",grpc_service="mwitkow.testproto.TestService",grpc_type="server_stream",le="+Inf"} 1
grpc_server_handling_seconds_sum{grpc_code="OK",grpc_method="PingList",grpc_service="mwitkow.testproto.TestService",grpc_type="server_stream"} 0.0003866430000000001
grpc_server_handling_seconds_count{grpc_code="OK",grpc_method="PingList",grpc_service="mwitkow.testproto.TestService",grpc_type="server_stream"} 1
````
