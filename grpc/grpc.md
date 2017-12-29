# Go gRPC
[官网](https://godoc.org/google.golang.org/grpc#pkg-constants)

## 常亮

```go
const (
    SupportPackageIsVersion3 = true
    SupportPackageIsVersion4 = true
    SupportPackageIsVersion5 = true
)
```

SupportPackageIsVersion变量是从生成的协议缓冲文件中引用的，以确保与所使用的gRPC版本兼容。 最新的支持包版本是5。

旧版本保持兼容性。 如果兼容性不能保持，可能会被删除。

这些常量不应该从任何其他代码引用。

```go
const PickFirstBalancerName = "pick_first"
```

PickFirstBalancerName是pick_first平衡器的名称。

```go
const Version = "1.9.0-dev"
```

Version是当前grpc版本

## 变量

```go
var (
    // ErrClientConnClosing表示该操作是非法的，因为ClientConn正在关闭。
    ErrClientConnClosing = errors.New("grpc: the client connection is closing")
    // ErrClientConnTimeout表示ClientConn无法在指定的超时时间内建立底层连接。 DEPRECATED：请使用context.DeadlineExceeded代替。
    ErrClientConnTimeout = errors.New("grpc: timed out when dialing")
)
```

```go
var DefaultBackoffConfig = BackoffConfig{
    MaxDelay:  120 * time.Second,
    baseDelay: 1.0 * time.Second,
    factor:    1.6,
    jitter:    0.2,
}
```
DefaultBackoffConfig使用在backoff中指定的值
https://github.com/grpc/grpc/blob/master/doc/connection-backoff.md

```go
var EnableTracing bool
```

EnableTracing控制是否使用golang.org/x/net/trace包来跟踪RPC。 这应该只在该程序发送或接收任何RPC之前设置。

```go
var ErrServerStopped = errors.New("grpc: the server has been stopped")
```

## func [Code](https://github.com/grpc/grpc-go/tree/master/rpc_util.go#L462)

```go
func Code(err error) codes.Code
```

如果rpc系统生成err，Code会返回err的错误代码。 否则，它将返回codes.Unknown。

不推荐使用：改用status.FromError和Code方法。

##  func [ErrorDesc](https://github.com/grpc/grpc-go/tree/master/rpc_util.go#L473)

```go
func ErrorDesc(err error) string
```

如果rpc系统产生err,ErrorDesc就返回err的错误描述.否则,当err为nil时,它返回err.Error()或空字符串

不推荐使用：改用status.FromError和Message方法.

## func [Errorf](https://github.com/grpc/grpc-go/tree/master/rpc_util.go#L484)

```go
func Errorf(c codes.Code, format string, a ...interface{}) error
```


Errorf返回一个包含错误代码和描述的error; 如果c是OK，Errorf返回nil。

不推荐使用：改用status.Errorf。

## func [Invoke](https://github.com/grpc/grpc-go/tree/master/call.go#L158)

```go
func Invoke(ctx context.Context, method string, args, reply interface{}, cc *ClientConn, opts ...CallOption) error
```

调用发生在线路上的rpc request,并在收到response后返回。这通常由生成的代码调用.

不建议使用:改用ClientConn.Invoke。

## func [MethodFromServerStream](https://github.com/grpc/grpc-go/tree/master/stream.go#L715)

```go
func MethodFromServerStream(stream ServerStream) (string, bool)
```

MethodFromServerStream返回输入流的方法字符串。 返回的字符串格式为“/service/method”。

## func [ NewLBBuilderWithFallbackTimeout](https://github.com/grpc/grpc-go/tree/master/grpclb.go#L100)

```go
func NewLBBuilderWithFallbackTimeout(fallbackTimeout time.Duration) balancer.Builder
```

NewLBBuilderWithFallbackTimeout使用给定的fallbackTimeout创建一个grpclb builder。 如果在fallbackTimeout内没有收到来自远程balancer的响应，将使用已解析地址列表中的后端地址。

只有在需要非默认回退超时时调用此函数。

## func [ SendHeader](https://github.com/grpc/grpc-go/tree/master/server.go#L1284)

```go
func SendHeader(ctx context.Context, md metadata.MD) error
```

SendHeader发送标题元数据。 最多可以调用一次。 由SetHeader()设置的提供的md和头文件将被发送。

## func [SetHeader](https://github.com/grpc/grpc-go/tree/master/server.go#L1271)

```go
func SetHeader(ctx context.Context, md metadata.MD) error
```

SetHeader设置header metadata。 多次调用时，所有提供的metadata将被合并。 发生以下情况时，所有metadata将被发送出去：

```go
- grpc.SendHeader() is called;
- The first response is sent out;
- An RPC status is sent out (error or success).
```

## func [SetTrailer](https://github.com/grpc/grpc-go/tree/master/server.go#L1301)

```go
func SetTrailer(ctx context.Context, md metadata.MD) error
```

SetTrailer设置当RPC返回时将被发送的尾部元数据。 当被多次调用时，所有提供的元数据将被合并。

## type [Address](https://github.com/grpc/grpc-go/tree/master/balancer.go#L36)

```go
type Address struct {
    // Addr是建立连接的服务器地址。
    Addr string
    // 元数据是与Addr相关的信息，可以用来做出负载均衡的决策。
    Metadata interface{}
}
```

Address代表客户端连接的服务器信息。 这是实验API，将来可能会改变或扩展。

## type [BackoffConfig](https://github.com/grpc/grpc-go/tree/master/backoff.go#L48)

```go
type BackoffConfig struct {
    // MaxDelay是补偿延迟的上限。
    MaxDelay time.Duration
    // 包含过滤或未导出的字段
}
```

BackoffConfig定义默认gRPC补偿策略的参数。

## type [Balancer](https://github.com/grpc/grpc-go/tree/master/balancer.go#L66)

```go
type Balancer interface {
    // 开始执行初始化工作来引导Balancer。 例如，此功能可能会启动名称解析并观察更新。 拨号时会被呼叫。
    Start(target string, config BalancerConfig) error
    // 向上通知Blancer，grpc与服务器有个基于addr地址的链接.一旦addr上的连接丢失或关闭，它就返回down
    // TODO: It is not clear how to construct and take advantage of the meaningful error
    // parameter for down. Need realistic demands to guide.
    Up(addr Address) (down func(error))
    // 获取与ctx对应的rpc的服务器地址
    // i) 如果它返回一个连接的地址，gRPC内部发送连接到这个地址的RPC;
    // ii)如果它返回正在建立连接的地址(由Notify(...)发起)但未连接，grpc内部会：
    //  * RPC失败: 如果RPC快速失败并且连接处于TransientFailure或Shutdown状态;
    //  * 发出RPC：在连接上
    // iii) 如果它返回一个不存在连接的地址,grpc内部会将其视为失败,并将使相应的rpc失败
    //
    // 因此,在编译自定义banlance时,推荐以下规则.如果opts.BlockingWait为true,则应该返回连接的地址或
    // 块(如果没有连接的地址).它应该考虑阻塞时ctx的超时或取.如果opts.BlockingWait为false(对于快速失败
    // 的RPC),它应该立即返回它通过Notify(...)通知的地址而不是阻塞
    //
    // 该函数返回一旦rpc完成或失败时调用的put。 put可以收集并向远程负载平衡器报告RPC状态。
    //
    //这个函数应该只返回错误，Balancer不能自行恢复。如果返回错误，RPC内部将会使RPC失败。
    Get(ctx context.Context, opts BalancerGetOptions) (addr Address, put func(), err error)
    // Notify 返回grpc内部使用的channel，观察grpc需要连接的地址.这些地址可能来自name resolver或者远程
    // 负载均衡器.grpc内部将把它与内部现有的连接地址进行比较.如果通知返回的address balancer不在现有的
    // 连接地址中,则grpc开始连接地址.如果现有连接得之中的地址不在通知列表中,则相应的连接将正常关闭.否则,
    // 不会有任何操作.请注意,地址片必须是应该连接的地址的完整列表.
    Notify() <-chan []Address
    // 关闭balancer
    Close() error
}
```
Balancer为RPC选择网络地址。 这是实验API，将来可能会改变或扩展。

### func [RoundRobin](https://github.com/grpc/grpc-go/tree/master/balancer.go#L138)

```go
func RoundRobin(r naming.Resolver) Balancer
```

RoundRobin返回一个轮训选择地址的Balancer。 它使用r来观察名称解析更新并相应地更新可用的地址。

## func [BalancerConfig](https://github.com/grpc/grpc-go/tree/master/balancer.go#L45)

```go
type BalancerConfig struct {
    // DialCreds 是Balancer实现用来拨号远程负载均衡服务器的传输证书
    // 如果不需要安全地与另一方通话，那么Balancer的实现可以忽略它。
    DialCreds credentials.TransportCredentials
    // Dialer 是Balancer实现可以用来拨号到远程负载平衡器服务器的自定义拨号程序。
    // 如果不需要与远程平衡器进行通信，则平衡器实现可以忽略这一点。
    Dialer func(context.Context, string) (net.Conn, error)
}
```

BalancerConfig指定了Balancer的配置。

## type [BalancerGetOptions](https://github.com/grpc/grpc-go/tree/master/balancer.go#L58)

```go
type BalancerGetOptions struct {
    // BlockingWait指定当没有连接地址时Get应该被阻塞。
    BlockingWait bool
}
```

BalancerGetOptions配置一个Get调用。 这是实验API，将来可能会改变或扩展。

## type [CallOption](https://github.com/grpc/grpc-go/tree/master/rpc_util.go#L143)

```go
type CallOption interface {
    // contains filtered or unexported methods
}
```

CallOption在启动之前配置呼叫，或者在呼叫完成后从呼叫中提取信息。

### func [FailFast](https://github.com/grpc/grpc-go/tree/master/rpc_util.go#L207)

```go
func FailFast(failFast bool) CallOption
```

FailFast配置在终端的连接或无法访问的服务器上尝试RPC时要执行的操作.如果failFast为true,rpc将立即失败,否则,rpc客户端将阻塞呼叫,直到连接可用或呼叫被取消或超时),并且如果由于瞬时错误而失败,则将重试呼叫.除非服务器指示数据不处理,grpc将不会重试.请参考https://github.com/grpc/grpc/blob/master/doc/wait-for-ready.md

默认情况下,rpc是"Fail Fast"

### func [Header](https://github.com/grpc/grpc-go/tree/master/rpc_util.go#L173)

```go
func Header(md *metadata.MD) CallOption
```

Header返回一个CallOptions，它检索一个unary RPC的Header Metadata。

### func [MaxCallRecvMsgSize](https://github.com/grpc/grpc-go/tree/master/rpc_util.go#L215)

```go
func MaxCallRecvMsgSize(s int) CallOption
```

MaxCallRecvMsgSize返回一个CallOption，它设置客户端可以接收的最大消息大小。

### func [MaxCallSendMsgSiz](https://github.com/grpc/grpc-go/tree/master/rpc_util.go#L223)

```go
func MaxCallSendMsgSize(s int) CallOption
```

MaxCallSendMsgSize返回一个CallOption,它设置客户端最大可发送消息的大小

### func [Peer](https://github.com/grpc/grpc-go/tree/master/rpc_util.go#L189)

```go
func Peer(peer *peer.Peer) CallOption
```

Peer返回一个CallOption,它检索一个unary rpc的对等信号.

### func [PerRPCCredentinals](https://github.com/grpc/grpc-go/tree/master/rpc_util.go#L232)

```go
func PerRPCCredentials(creds credentials.PerRPCCredentials) CallOption
```

PerRPCCredentials返回一个CallOption，为一次调用设置credentials.PerRPCCredentials。

### func [Trailer](https://github.com/grpc/grpc-go/tree/master/rpc_util.go#L181)

```go
func Trailer(md *metadata.MD) CallOption
```

Trailer返回一个CallOptions，它检索一个一元RPC的尾部元数据。

### func [UseCompressor](https://github.com/grpc/grpc-go/tree/master/rpc_util.go#L244)

```go
func UseCompressor(name string) CallOption
```

UseCompressor返回一个CallOption，它设置发送请求时使用的Compressor(压缩器)。 如果Compressor也设置，UseCompressor具有更高的优先级。
这个API是实验性的。

## type [ClientConn](https://github.com/grpc/grpc-go/tree/master/clientconn.go#L603)

```go
type ClientConn struct {
    // contains filtered or unexported fields
}
```

ClientConn表示到RPC服务器的客户端连接。

### func [Dial](https://github.com/grpc/grpc-go/tree/master/clientconn.go#L402)

```go
func Dial(target string, opts ...DialOption) (*ClientConn, error)
```

Dial（拨号器）创建到给定目标的客户端连接。

### func [DialContext](https://github.com/grpc/grpc-go/tree/master/clientconn.go#L410)

```go
func DialContext(ctx context.Context, target string, opts ...DialOption) (conn *ClientConn, err error)
```

DialContext创建给定目标的客户连接. 可以使用ctx取消或终止挂起的链接.一旦这个函数返回,ctx的取消和到期将是空操作.次函数返回后,用户应调用ClientConn.Close来终止所有挂起的操作

### func [(*ClientConn)Close](https://github.com/grpc/grpc-go/tree/master/clientconn.go#L938)

```go
func (cc *ClientConn) Close() error
```

关闭ClientConn和所有底层连接

### func [(*ClientConn)GetMethodConfig](https://github.com/grpc/grpc-go/tree/master/clientconn.go#L880)

```go
func (cc *ClientConn) GetMethodConfig(method string) MethodConfig
```

GetMethodConfig获取输入方法的配置方法。 如果输入方法（即/service/method）完全匹配，则返回相应的MethodConfig。 如果输入方法没有完全匹配，则查找服务下的默认配置（即/service/）。 如果服务有一个默认的MethodConfig，我们返回它。 否则，我们返回一个空的MethodConfig。

### func [(*ClientConn)GetState](https://github.com/grpc/grpc-go/tree/master/clientconn.go#L647)

```go
func (cc *ClientConn) GetState() connectivity.State
```

GetState返回ClientConn的连接状态。 这是一个实验性的API。

### func [(*ClientConn)Invoke](https://github.com/grpc/grpc-go/tree/master/call.go#L147)

```go
func (cc *ClientConn) Invoke(ctx context.Context, method string, args, reply interface{}, opts ...CallOption) error
```

Invoker发送在线上的RPC request,并在收到response后返回.这通常由生成的代码调用。

### func [(*ClientConn)NewStream](https://github.com/grpc/grpc-go/tree/master/stream.go#L99)

```go
func (cc *ClientConn) NewStream(ctx context.Context, desc *StreamDesc, method string, opts ...CallOption) (ClientStream, error)
```

NewStream为客户端创建一个新的Stream。 这通常由生成的代码调用。

### func [(*ClientConn)WaitForStateChange](https://github.com/grpc/grpc-go/tree/master/clientconn.go#L632)

```go
func (cc *ClientConn) WaitForStateChange(ctx context.Context, sourceState connectivity.State) bool
```

WaitForStateChange等待连接。ClientConn的状态从sourceState或ctx更改过期。 在前一种情况下返回一个真值，在后者中返回假。 这是一个实验性的API。

## type [ClientStream](https://github.com/grpc/grpc-go/tree/master/stream.go#L78)

```go
type ClientStream interface {
    // 如果有数据Header会返回头部元数据
    // 如果元数据还没有准备好读取,它将被阻塞.
    Header() (metadata.MD, error)
    // 如果有数据,Trailer返回来自服务器的尾部元数据
    // 它只能在stream.CloseAndRecv返回之后调用,否则stream.Recv返回一个非nil error(包括io.EOF)
    Trailer() metadata.MD
    // CloseSend关闭流的发送指引。 当㓟非nil错误时，它关闭流。
    CloseSend() error
    // 当发送请求发生错误时，Stream.SendMsg（）可能会返回非nil错误。 返回的错误表示当前发送的状态，而不
    // 是RPC的最终状态。
    // 如果你关心RPC的状态，一般调用Stream.RecvMsg（）来获得最终状态
    Stream
}
```

ClientStream定义了客户端流必须满足的接口。

### func [NewClientStream](https://github.com/grpc/grpc-go/tree/master/stream.go#L110)

```go
func NewClientStream(ctx context.Context, desc *StreamDesc, cc *ClientConn, method string, opts ...CallOption) (ClientStream, error)
```

NewClientStream为客户端创建一个新的Stream。 这通常由生成的代码调用。

DEPRECATED：改用ClientConn.NewStream。

## type [Codes](https://github.com/grpc/grpc-go/tree/master/codec.go#L31)

```go
type Codec interface {
    // Marshal returns the wire format of v.
    Marshal(v interface{}) ([]byte, error)
    // Unmarshal parses the wire format into v.
    Unmarshal(data []byte, v interface{}) error
    // String returns the name of the Codec implementation. The returned
    // string will be used as part of content type in transmission.
    String() string
}
```

Codec定义了gRPC用于编码和解码消息的接口。 请注意，此接口的实现必须是线程安全的; 编解码器的方法可以从并发的goroutines中调用。

## type [Compressor](https://github.com/grpc/grpc-go/tree/master/rpc_util.go#L43)

```go
type Compressor interface {
    // 把p压缩成w
    Do(w io.Writer, p []byte) error
    //Type返回Compressor使用的压缩算法。
    Type() string
}
```

Compressor定义了gRPC用来压缩消息的接口。

### func [NewGZIPCompressor](https://github.com/grpc/grpc-go/tree/master/rpc_util.go#L55)

```go
func NewGZIPCompressor() Compressor
```

NewGZIPCompressor创建一个基于GZIP的压缩器。

## type [Decompressor](https://github.com/grpc/grpc-go/tree/master/rpc_util.go#L80)

```go
type Decompressor interface {
    // 从r读取数据并解压缩它们。
    Do(r io.Reader) ([]byte, error)
    // Type返回Decompressor使用的压缩算法。
    Type() string
}
```

Decompressor定义gRPC用于解压缩消息的接口。

### func [NewGZIPDecompressor](https://github.com/grpc/grpc-go/tree/master/rpc_util.go#L92)

```go
func NewGZIPDecompressor() Decompressor
```

NewGZIPDecompressor在GZIP的基础上创建一个解压缩器.

## type [DialOption](https://github.com/grpc/grpc-go/tree/master/clientconn.go#L114)

```go
type DialOption func(*dialOptions)
```

DialOption配置我们如何建立连接

### func [FailOnNonTempDialError](https://github.com/grpc/grpc-go/tree/master/clientconn.go#L358)

```go
func FailOnNonTempDialError(f bool) DialOption
```

FailOnNonTempDialError返回一个DialOption，用于指定gRPC是否因non-temporary dial错误而失败。如果f为true,并且dialer返回非临时性错误,则grpc的网络地址的连接将失败,并且不会尝试重新连接.FailOnNonTempDialError的默认值是false。 这是一个实验性的API。

### func [WithAuthority](https://github.com/grpc/grpc-go/tree/master/clientconn.go#L395)

```go
func WithAuthority(a string) DialOption
```

WithAuthority返回一个DialOption，它指定要用作：authority伪标题的值。 此值仅适用于WithInsecure，如果存在TransportCredentials，则不起作用。

### func [WithBackoffConfig](https://github.com/grpc/grpc-go/tree/master/clientconn.go#L267)

```go
func WithBackoffConfig(b BackoffConfig) DialOption
```

WithBackoffConfig配置dialer在连接失败后使用提供的补偿参数。

使用WithBackoffMaxDelay，直到BackoffConfig上的更多参数打开使用。

### func [WithBackoffMasDelay](https://github.com/grpc/grpc-go/tree/master/clientconn.go#L258)

```go
func WithBackoffMaxDelay(md time.Duration) DialOption
```

WithBackoffMaxDelay配置拨号程序在失败连接尝试后补偿时使用提供的最大延迟。

### func [WithBalancer](https://github.com/grpc/grpc-go/tree/master/clientconn.go#L206)

```go
func WithBalancer(b Balancer) DialOption
```

WithBalancer返回一个DialOption，它用v1 API设置一个负载均衡器。 如果指定了DialOption，名称解析器将被忽略。

弃用：在balancer package和WithBalancerName中使用新的Balancer API。

### func [WithBalancerName](https://github.com/grpc/grpc-go/tree/master/clientconn.go#L222)

```go
func WithBalancerName(balancerName string) DialOption
```

WithBalancerName设置ClientConn将被初始化的balancer。 使用balancerName注册的balancer。 如果没有balancer被balancerName注册，这个函数会发生混乱。

balancer不能被service config指定的balancer选项覆盖。

这是一个实验性的API。

### func [WithBlock](https://github.com/grpc/grpc-go/tree/master/clientconn.go#L287)

```go
func WithBlock() DialOption
```

WithBlock返回一个DialOption，它使dial block的调用者，直到底层连接启动。 没有这个，dial立即返回，连接服务器发生在后台。

### func [WithCodes](https://github.com/grpc/grpc-go/tree/master/clientconn.go#L170)

```go
func WithCodec(c Codec) DialOption
```

WithCodec返回一个DialOption，它为消息的编解码设置一个编解码器。

### fun [WithCompressor](https://github.com/grpc/grpc-go/tree/master/clientconn.go#L181)

```go
func WithCompressor(cp Compressor) DialOption
```

WithCompressor返回一个DialOption，它设置一个Compressor用于消息压缩。 它的优先级比UseCompressor CallOption设置的压缩器低。

弃用：改用UseCompressor。

### func [WithDecompressor](https://github.com/grpc/grpc-go/tree/master/clientconn.go#L196)

```go
func WithDecompressor(dc Decompressor) DialOption
```

WithDecompressor返回一个DialOption，它设置一个Decompressor用于传入的消息解压缩。 如果传入的响应消息使用解压缩程序的Type（）进行编码，则将使用它。 否则，将使用消息编码来查找通过encoding.RegisterCompressor注册的压缩器，然后将其用于解压缩消息。 如果没有压缩器注册编码，则会返回未实现状态错误。

弃用：改为使用encoding.RegisterCompressor。

### func [WithDefaultCallOptions](https://github.com/grpc/grpc-go/tree/master/clientconn.go#L163)

```go
func WithDefaultCallOptions(cos ...CallOption) DialOption
```

WithDefaultCallOptions返回一个DialOption，它为连接上的调用设置默认的CallOptions。

### func [WithDialer](https://github.com/grpc/grpc-go/tree/master/clientconn.go#L335)

```go
func WithDialer(f func(string, time.Duration) (net.Conn, error)) DialOption
```

WithDialer返回一个DialOption，它指定一个用于拨号网络地址的函数。 如果FailOnNonTempDialError（）设置为true，并且f返回error，gRPC将检查error的Temporary（）方法，以决定是否应该尝试重新连接到网络地址。

### func [WithInitialConnWindowSize](https://github.com/grpc/grpc-go/tree/master/clientconn.go#L151)

```go
func WithInitialConnWindowSize(s int32) DialOption
```

WithInitialConnWindowSize返回一个DialOption，它设置连接上初始窗口大小的值。 窗口大小的下限是64K，小于这个值的任何值都将被忽略。

### func [WithInitialWindowZise](https://github.com/grpc/grpc-go/tree/master/clientconn.go#L143)

```go
func WithInitialWindowSize(s int32) DialOption
```

WithInitialWindowSize返回一个DialOption，它设置流上初始窗口大小的值。 窗口大小的下限是64K，小于这个值的任何值都将被忽略。

### func [WithInsecure](https://github.com/grpc/grpc-go/tree/master/clientconn.go#L295)

```go
func WithInsecure() DialOption
```

WithInsecure返回一个DialOption，为此ClientConn禁用传输安全性。 请注意，除非设置了“WithInsecure”，否则需要传输安全性。

### func [WithKeepaliveParams](https://github.com/grpc/grpc-go/tree/master/clientconn.go#L372)

```go
func WithKeepaliveParams(kp keepalive.ClientParameters) DialOption
```

WithKeepaliveParams返回一个DialOption，它为客户端传输指定keepalive参数。

### func [WithMaxMsgSize](https://github.com/grpc/grpc-go/tree/master/clientconn.go#L158)

```go
func WithMaxMsgSize(s int) DialOption
```
WithMaxMsgSize返回一个DialOption，它设置客户端可以接收的最大消息大小。 弃用：改用WithDefaultCallOptions（MaxCallRecvMsgSize（s））。

### func [WithPerRPCCredentials](https://github.com/grpc/grpc-go/tree/master/clientconn.go#L311)

```go
func WithPerRPCCredentials(creds credentials.PerRPCCredentials) DialOption
```

WithPerRPCCredentials返回一个DialOption，它设置凭证并在每个出站RPC上放置auth状态。

### func [WithReadBufferSize](https://github.com/grpc/grpc-go/tree/master/clientconn.go#L135)

```go
func WithReadBufferSize(s int) DialOption
```

WithResolverUserOptions返回一个DialOption，它设置了解析器的BuildOption的UserOptions字段。

### func [WithServiceConfig](https://github.com/grpc/grpc-go/tree/master/clientconn.go#L250)

```go
func WithServiceConfig(c <-chan ServiceConfig) DialOption
```

WithServiceConfig返回一个DialOption，它有一个读取服务配置的通道。 DEPRECATED：服务配置应通过名称解析器接收，如在此处指定。https://github.com/grpc/grpc/blob/master/doc/service_config.md

### func [WithStatsHandler](https://github.com/grpc/grpc-go/tree/master/clientconn.go#L347)

```go
func WithStatsHandler(h stats.Handler) DialOption
```

WithStatsHandler返回一个DialOption，它为此ClientConn中的所有RPC和底层网络连接指state handler。

### func [WithStreamInerceptor](https://github.com/grpc/grpc-go/tree/master/clientconn.go#L386)

```go
func WithStreamInterceptor(f StreamClientInterceptor) DialOption
```

WithStreamInterceptor返回一个DialOption，指定用于流式RPC的拦截器。

### func [WithTimeout](https://github.com/grpc/grpc-go/tree/master/clientconn.go#L320)

```go
func WithTimeout(d time.Duration) DialOption
```

WithTimeout返回一个DialOption，用于配置最初拨打ClientConn的超时时间。 当且仅当WithBlock（）存在时有效。 弃用：改为使用DialContext和context.WithTimeout。

### func [WithTransportCredentials](https://github.com/grpc/grpc-go/tree/master/clientconn.go#L303)

```go
func WithTransportCredentials(creds credentials.TransportCredentials) DialOption
```

WithTransportCredentials返回一个DialOption，用于配置连接级别的安全证书（例如TLS / SSL）。

### func [WithUnaryInterceptor](https://github.com/grpc/grpc-go/tree/master/clientconn.go#L379)

```go
func WithUnaryInterceptor(f UnaryClientInterceptor) DialOption
```

WithUnterInterceptor返回一个DialOption，它指定unary RPC的拦截器。

### func [WithUserAgent](https://github.com/grpc/grpc-go/tree/master/clientconn.go#L365)

```go
func WithUserAgent(s string) DialOption
```

WithUserAgent返回一个DialOption，它指定所有RPC的用户代理字符串。

### func [WithWaitForHandsShake](https://github.com/grpc/grpc-go/tree/master/clientconn.go#L119)

```go
func WithWaitForHandshake() DialOption
```

在将RPC分配给连接之前，使用WaitForHandshake块直到从服务器接收到初始设置帧。 实验API。

### func [WithWriteBufferSize](https://github.com/grpc/grpc-go/tree/master/clientconn.go#L127)

```go
func WithWriteBufferSize(s int) DialOption
```

WithWriteBufferSize允许您设置写入缓冲区的大小，这决定了在写入数据之前可以批量处理多少数据。

## type [EmptyCallOption](https://github.com/grpc/grpc-go/tree/master/rpc_util.go#L156)

```go
type EmptyCallOption struct{}
```

EmptyCallOption不会改变呼叫配置。 它可以嵌入到另一个结构中，以携带拦截器使用的卫星数据。

## type [MethodConfig](https://github.com/grpc/grpc-go/tree/master/service_config.go#L38)

```go
type MethodConfig struct {
    // WaitForReady指示发送到此方法的RPC是否应该等待连接准备就绪（！failfast）。 通过gRPC客户端API指定
    //的值将覆盖此处设置的值。
    WaitForReady *bool
    // Timeout是发送给此方法的PRC的默认超时值.使用的实际deadline是此处制定的值得最小值以及应用
    //程序通过grpc客户端api设置的值。如果其中一个没有设置,那么另一个将被设置.如果都没有设置,那么prc没
    //有deadline
    Timeout *time.Duration
    // MaxReqSize是流（CLIENT - >Server）中单个请求所允许的最大有效负载大小（以字节为单位）。
    // 测量的大小是每个消息在压缩之后(但在stream压缩之前)的以字节为单位的序列化payload。
    //实际使用的值是此处指定的值的最小值和应用程序通过gRPC客户端API设置的值。 如果其中一个没有设置，
    //那么另一个将被使用。 如果两者均未设置，则使用内置的默认值。
    MaxReqSize *int
    // MaxRespSize是流（server - >client）中单个响应的最大允许有效负载大小（以字节为单位）。
    MaxRespSize *int
}
```

MethodConfig定义了服务提供者为特定方法推荐的配置。 DEPRECATED：用户不应该使用这个结构。 服务配置应通过名称解析器接收，如https：//github.com/grpc/grpc/blob/master/doc/service_config.md

## type [MethodDesc](https://github.com/grpc/grpc-go/tree/master/server.go#L61)

```go
type MethodDesc struct {
    MethodName string
    Handler    methodHandler
}
```

MethodDesc表示RPC服务的方法规范。

## type [MethodInfo](https://github.com/grpc/grpc-go/tree/master/server.go#L406)

```go
type MethodInfo struct {
    // Name is the method name only, without the service name or package name.
    Name string
    // IsClientStream indicates whether the RPC is a client streaming RPC.
    IsClientStream bool
    // IsServerStream indicates whether the RPC is a server streaming RPC.
    IsServerStream bool
}
```

MethodInfo包含RPC的信息，包括其方法名称和类型。

## type [Server](https://github.com/grpc/grpc-go/tree/master/server.go#L87)

```go
type Server struct {
    // contains filtered or unexported fields
}
```

Server是提供RPC请求的gRPC服务器。

### func [NewServer](https://github.com/grpc/grpc-go/tree/master/server.go#L325)

```go
func NewServer(opt ...ServerOption) *Server
```

NewServer创建一个没有注册服务的gRPC server，并且还没有开始接受请求。

### func [(*Server)GetServiceInfo](https://github.com/grpc/grpc-go/tree/master/server.go#L424)

```go
func (s *Server) GetServiceInfo() map[string]ServiceInfo
```

GetServiceInfo从service name返回一个map到ServiceInfo。 service name包含package名称，格式为<package>.<service>。

### func [(*Server) GracefulStop](https://github.com/grpc/grpc-go/tree/master/server.go#L1215)

```go
func (s *Server) GracefulStop()
```

GracefulStop优雅地停止gRPC服务器。 它会阻止服务器接受新的连接和RPC，并阻止所有挂起的RPC完成。

### func [(*Server) RegisterService](https://github.com/grpc/grpc-go/tree/master/server.go#L369)

```go
func (s *Server) RegisterService(sd *ServiceDesc, ss interface{})
```

RegisterService将服务及其实现注册到gRPC服务器。 它是从IDL生成的代码中调用的。 这必须在调用Serve之前调用。

### func [(*Server) Serve](https://github.com/grpc/grpc-go/tree/master/server.go#L468)

```go
func (s *Server) Serve(lis net.Listener) error
```

Serve接受Listener上的传入的连接，为每个服务器创建一个新的ServerTransport和服务配置。 service goroutine读取gRPC请求，然后调用注册的handler来回复它们。 当传入的参数lis.Accept fail(致命错误)时Server会返回。当此方法返回时，lis将被关闭。 Serve将返回一个非nil错误，除非Stop或GracefulStop被调用。

### func [(*Server)ServerHTTP](https://github.com/grpc/grpc-go/tree/master/server.go#L697)

```go
func (s *Server) ServeHTTP(w http.ResponseWriter, r *http.Request)
```

ServeHTTP通过在gRPC server中查找所请求的gRPC方法来响应gRPC请求r来实现Go标准库的http.Handler接口。

提供的HTTP请求必须已经到达HTTP/2连接。 使用Go标准库的Server时，实际上意味着请求也必须通过TLS到达。

要在gRPC和现有的http.Handler之间共享一个端口（例如https的443），请使用root http.Handler，例如：

```go
if r.ProtoMajor == 2 && strings.HasPrefix(
	r.Header.Get("Content-Type"), "application/grpc") {
	grpcServer.ServeHTTP(w, r)
} else {
	yourMux.ServeHTTP(w, r)
}
```

注意ServeHTTP使用Go的HTTP/2 server实现，它与grpc-go的HTTP/2 server完全分离。 两种路径的性能和功能可能有所不同。 ServeHTTP不支持通过grpc-go的HTTP/2 server提供的一些gRPC功能，目前它是实验性的，可能会有变化。

### func [(*Server)Stop](https://github.com/grpc/grpc-go/tree/master/server.go#L1176)

```go
func (s *Server) Stop()
```

Stop停止gRPC服务器。 它立即关闭所有打开的连接和监听器。 它将取消服务器端的所有活动RPC，并且客户端上相应的挂起RPC将通过连接错误得到通知。

## type [ServerOption](https://github.com/grpc/grpc-go/tree/master/server.go#L136)

```go
type ServerOption func(*options)
```

ServerOption设置诸如凭证，编解码器和keepalive参数等选项。

### func [ConnectionTimeout](https://github.com/grpc/grpc-go/tree/master/server.go#L317)

```go
func ConnectionTimeout(d time.Duration) ServerOption
```

ConnectionTimeout返回一个ServerOption，为所有新的连接设置连接建立的超时（直到并包括HTTP/2握手）。 如果没有设置，默认值是120秒。 零或负值将导致立即超时。

这个API是实验性的。

### func [Creds](https://github.com/grpc/grpc-go/tree/master/server.go#L246)

```go
func Creds(c credentials.TransportCredentials) ServerOption
```

Creds返回一个为服务器连接设置凭据的ServerOption。

### func [CustomCodec](https://github.com/grpc/grpc-go/tree/master/server.go#L185)

```go
func CustomCodec(codec Codec) ServerOption
```

CustomCodec返回一个ServerOption，它为消息编解码设置一个编解码器。

### func [InTapHandle](https://github.com/grpc/grpc-go/tree/master/server.go#L277)

```go
func InTapHandle(h tap.ServerInHandle) ServerOption
```

InTapHandle返回一个ServerOption，它设置所有要创建的服务器传输的tap handler。 只能安装一个。

### func [InitialConnWindowSize](https://github.com/grpc/grpc-go/tree/master/server.go#L164)

```go
func InitialConnWindowSize(s int32) ServerOption
```

InitialConnWindowSize返回一个为连接设置窗口大小的ServerOption。 窗口大小的下限是64K，小于这个值的任何值都将被忽略。

### func [InitialWindowSize](https://github.com/grpc/grpc-go/tree/master/server.go#L156)

```go
func InitialWindowSize(s int32) ServerOption
```

InitialWindowSize返回一个ServerOption，它设置流的窗口大小。 窗口大小的下限是64K，小于这个值的任何值都将被忽略。

### func [KeepaliveEnforcementPolicy](https://github.com/grpc/grpc-go/tree/master/server.go#L178)

```go
func KeepaliveEnforcementPolicy(kep keepalive.EnforcementPolicy) ServerOption
```

KeepaliveEnforcementPolicy返回一个ServerOption，为Server设置Keepalive执行策略。

### func [KeepaliveParams](https://github.com/grpc/grpc-go/tree/master/server.go#L171)

```go
func KeepaliveParams(kp keepalive.ServerParameters) ServerOption
```

KeepaliveParams返回一个ServerOption，为服务器设置keepalive和max-age参数。

### func [MaxConcurrentStreams](https://github.com/grpc/grpc-go/tree/master/server.go#L239)

```go
func MaxConcurrentStreams(n uint32) ServerOption
```

### func [MaxMsgSize](https://github.com/grpc/grpc-go/tree/master/server.go#L217)

```go
func MaxMsgSize(m int) ServerOption
```

MaxMsgSize返回ServerOption以设置服务器可以接收的最大消息大小（以字节为单位）。 如果没有设置，gRPC使用默认限制。 弃用：改用MaxRecvMsgSize。

### func [MaxRecvMsgSize](https://github.com/grpc/grpc-go/tree/master/server.go#L223)

```go
func MaxRecvMsgSize(m int) ServerOption
```

MaxRecvMsgSize返回ServerOption来设置服务器可以接收的最大消息大小（以字节为单位）。 如果没有设置，gRPC使用默认的4MB。

### func [MaxSendMsgSize](https://github.com/grpc/grpc-go/tree/master/server.go#L231)

```go
func MaxSendMsgSize(m int) ServerOption
```

MaxSendMsgSize返回ServerOption来设置服务器可以发送的最大消息大小（以字节为单位）。 如果没有设置，gRPC使用默认的4MB。

### fun [RPCCompressor](https://github.com/grpc/grpc-go/tree/master/server.go#L198)

```go
func RPCCompressor(cp Compressor) ServerOption
```

RPCCompressor返回一个ServerOption,它设置出站消息的Compressor。为了向后兼容,无论传入的消息压缩如何,所有出站消息都将用此Compressor发送.默认情况下,server 消息将使用发送request消息相同的Compressor

弃用：改为使用encoding.RegisterCompressor。

###  func [RPCDecompressor](https://github.com/grpc/grpc-go/tree/master/server.go#L209)

```go
func RPCDecompressor(dc Decompressor) ServerOption
```

RPCDecompressor返回一个ServerOption，为入站消息设置一个解压缩器。 它比通过encoding.RegisterCompressor注册的解压缩程序具有更高的优先级。

弃用：改为使用encoding.RegisterCompressor。

### func [ReadBufferSiz](https://github.com/grpc/grpc-go/tree/master/server.go#L148)

```go
func ReadBufferSize(s int) ServerOption
```

ReadBufferSize允许您设置读取缓冲区的大小，这决定了一个读取系统调用最多可以读取多少数据。

### func [StatsHandler](https://github.com/grpc/grpc-go/tree/master/server.go#L287)

```go
func StatsHandler(h stats.Handler) ServerOption
```

StatsHandler返回一个ServerOption，它为服务器设置stats handler。

### func [StreamInterceptor](https://github.com/grpc/grpc-go/tree/master/server.go#L266)

```go
func StreamInterceptor(i StreamServerInterceptor) ServerOption
```

StreamInterceptor返回一个ServerOption，它为服务器设置StreamServerInterceptor。 只能安装一个流拦截器。

### func [UnaryInterceptor](https://github.com/grpc/grpc-go/tree/master/server.go#L255)

```go
func UnaryInterceptor(i UnaryServerInterceptor) ServerOption
```

UnaryInterceptor返回一个ServerOption，为服务器设置UnaryServerInterceptor。 只能安装一个unary拦截器。 可以在调用者处实现多个拦截器（例如链接）的构建。

### func [UnknownServiceHandler](https://github.com/grpc/grpc-go/tree/master/server.go#L299)

```go
func UnknownServiceHandler(streamHandler StreamHandler) ServerOption
```

UnKnownServiceHandler返回一个ServerOption，允许添加一个自定义的未知服务处理器.所提供的方法是一个双向流RPC service handler,只要接收到未注册的服务或方法的请求,就会被调用,而不是"UnImplements"grpc错误. handler可以完全访问请求的上线和Stream,以及调用可以绕过拦截器。

### func [WriteBufferSize](https://github.com/grpc/grpc-go/tree/master/server.go#L140)

```go
func WriteBufferSize(s int) ServerOption
```

WriteBufferSize允许您设置写入缓冲区的大小，这决定了在写入数据之前可以批量处理多少数据。

## type [ServerStream](https://github.com/grpc/grpc-go/tree/master/stream.go#L573)

```go
type ServerStream interface {
    // SetHeader设置header metadata. 它可能被多次调用.
    // 当多次调用时,所有提供的metadata将被合并.
    //发生以下情况时，所有metadata将被发送出去:
    //  - ServerStream.SendHeader()被调用;
    //  - 第一个RESPONSE被发送出去;
    //  - 一个RPC状态被发送出去(error or success).
    SetHeader(metadata.MD) error
    // SendHeader 发送header metadata.
    // 参数提供的md和SetHeader()设置的head matadata🏆发送
    // 多次调用时会失败
    SendHeader(metadata.MD) error
    // SetTrailer设置随RPC状态一起发送的trail metadata。
    // 当多次调用时,所有提供的metadata将被合并
    SetTrailer(metadata.MD)
    Stream
}
```

## type [ServiceConfig](https://github.com/grpc/grpc-go/tree/master/service_config.go#L65)

```go
type ServiceConfig struct {
    // LB是服务提供商推荐的load balancer。 通过grpc.WithBalancer指定的balancer将覆盖此。
    LB  *string
    // Methods 包含此服务中方法的map.如果map中的方法(即/service/method)完全匹配,则使用相应的
    // MethodConfig。如果没有完全匹配,请查找服务的默认配置(/service/)，如果存在,则使用相应的
    // MethodConfig;否则,该方法没有MethodConfig可以使用.
    Methods map[string]MethodConfig
}
```

ServiceConfig由服务提供者提供，并包含有关连接到服务的客户端如何表现的参数。 DEPRECATED：用户不应该使用这个结构体。 服务配置应通过name resolver接收，如https：//github.com/grpc/grpc/blob/master/doc/service_config.md

## type [ServiceDesc](https://github.com/grpc/grpc-go/tree/master/server.go#L67)

```go
type ServiceDesc struct {
    ServiceName string
    // 指向服务接口的指针。 用于检查用户提供的实现是否满足接口要求。
    HandlerType interface{}
    Methods     []MethodDesc
    Streams     []StreamDesc
    Metadata    interface{}
}
```

ServiceDesc表示RPC服务的规范。

## type [ServiceInfo](https://github.com/grpc/grpc-go/tree/master/server.go#L416)

```go
type ServiceInfo struct {
    Methods []MethodInfo
    // Metadata 是在注册服务时在ServiceDesc中指定的metadata
    Metadata interface{}
}
```

ServiceInfo包含unary RPC方法信息，stream RPC method信息和service的metadata.

## type [Stream](https://github.com/grpc/grpc-go/tree/master/stream.go#L54)

```go
type Stream interface {
    // Context 返回stream的上下文
    Context() context.Context
    // SendMsg是阻塞的,直到它发送m，stream完成或中断.发送error时,它会中断stream,
    // 并返回客户端上的RPC状态.在服务端,它只是将error返回给调用者.sendMsg由生成的代码调用
    // 当用户真正需要的时候,用户可以直接调用SendMsg。有一个调用SendMsg的goroutine和另一个goroutine同
    // 时在一个stream上调用recvMsg是安全的。但是在不同的goroutine中的同一个stream上调用SendMsg是
    // 不安全的。
    SendMsg(m interface{}) error
    // RecvMsg是阻塞的,直到收到消息或stream完成.在客户端,当stream完成时返回io.EOF.
    // 发生任何其他error时，它终端stream并返回一个RPC状态.在服务端,它只是将错误返回给调用者.
    RecvMsg(m interface{}) error
}
```

Stream定义客户端或服务器流必须满足的通用接口。

## type [StreamClientInterceptor](https://github.com/grpc/grpc-go/tree/master/interceptor.go#L39)

```go
type StreamClientInterceptor func(ctx context.Context, desc *StreamDesc, cc *ClientConn, method string, streamer Streamer, opts ...CallOption) (ClientStream, error)
```

StreamClientInterceptor拦截ClientStream的创建。 它可能会返回一个自定义的ClientStream来拦截所有的I/O操作。 streamer是创建ClientStream的处理器，拦截器负责调用它。 这是一个实验性的API。

## type [StreamDesc](https://github.com/grpc/grpc-go/tree/master/stream.go#L44)

```go
type StreamDesc struct {
    StreamName string
    Handler    StreamHandler

    // At least one of these is true.
    ServerStreams bool
    ClientStreams bool
}
```

StreamDesc表示流式RPC服务的方法规范。

## type [StreamHandler](https://github.com/grpc/grpc-go/tree/master/stream.go#L41)

```go
type StreamHandler func(srv interface{}, stream ServerStream) error
```

StreamHandler定义gRPC服务器调用的处理程序来完成stream RPC的执行。

## type [StreamServerInfo](https://github.com/grpc/grpc-go/tree/master/interceptor.go#L62)

```go
type StreamServerInfo struct {
    // FullMethod is the full RPC method string, i.e., /package.service/method.
    FullMethod string
    // IsClientStream indicates whether the RPC is a client streaming RPC.
    IsClientStream bool
    // IsServerStream indicates whether the RPC is a server streaming RPC.
    IsServerStream bool
}
```

StreamServerInfo包含有关服务器端的流式RPC的各种信息。 所有per-rpc信息都可能被拦截器突变。

## type [StreamServerInterceptor](https://github.com/grpc/grpc-go/tree/master/interceptor.go#L75)

```go
type StreamServerInterceptor func(srv interface{}, ss ServerStream, info *StreamServerInfo, handler StreamHandler) error
```

StreamServerInterceptor提供了一个钩子来拦截服务器上流式RPC的执行。 info包含拦截器可以操作的这个RPC的所有信息。 处理程序是服务方法的实现。 拦截器负责调用处理程序来完成RPC。

## type [Stream](https://github.com/grpc/grpc-go/tree/master/interceptor.go#L34)

```go
type Streamer func(ctx context.Context, desc *StreamDesc, cc *ClientConn, method string, opts ...CallOption) (ClientStream, error)
```

Stream Client被Stream ClientInterceptor调用来创建一个ClientStream。

## type [UnaryClientInterceptor](https://github.com/grpc/grpc-go/tree/master/interceptor.go#L31)

```go
type UnaryClientInterceptor func(ctx context.Context, method string, req, reply interface{}, cc *ClientConn, invoker UnaryInvoker, opts ...CallOption) error
```

UnaryClientInterceptor拦截客户端上unary RPC的执行。 invoker是完成RPC的处理程序，拦截器负责调用它。 这是一个实验性的API。

## type [UnaryHandler](https://github.com/grpc/grpc-go/tree/master/interceptor.go#L52)

```go
type UnaryHandler func(ctx context.Context, req interface{}) (interface{}, error)
```

UnaryHandler定义由UnaryServerInterceptor调用的处理程序来完成unary RPC的正常执行。

## type [UnaryInoker](https://github.com/grpc/grpc-go/tree/master/interceptor.go#L26)

```go
type UnaryInvoker func(ctx context.Context, method string, req, reply interface{}, cc *ClientConn, opts ...CallOption) error
```

UnaryClientInterceptor调用UnaryInvoker来完成RPC。

## type [UnaryServerInfo](https://github.com/grpc/grpc-go/tree/master/interceptor.go#L43)

```go
type UnaryServerInfo struct {
    // Server是用户提供的服务实现。 这是只读的。
    Server interface{}
    // FullMethod is the full RPC method string, i.e., /package.service/method.
    FullMethod string
}
```

UnaryServerInfo由关于服务器端的unary RPC的各种信息组成。 所有per-rpc信息都可能被拦截器突变。

## type [UnaryServerInterceptor](https://github.com/grpc/grpc-go/tree/master/interceptor.go#L58)

```go
type UnaryServerInterceptor func(ctx context.Context, req interface{}, info *UnaryServerInfo, handler UnaryHandler) (resp interface{}, err error)
```

UnaryServerInterceptor提供了一个拦截服务器上unary RPC执行的钩子。 info包含拦截器可以操作的这个RPC的所有信息。 处理程序是服务方法实现的包装。 拦截器负责调用处理程序来完成RPC。

