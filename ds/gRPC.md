https://doc.oschina.net/grpc?t=56831
Google开源框架

gRPC允许你为RPC ( Remote Procedure Call)定义请求和响应，然后gRPC会帮你处理一切剩余问题。

速度快，执行效率高，基于HTTP/2构建，低延迟，支持流，与开发语言无关，并且可以很简单的插入身份认证、负载均衡、日志和监控等功能。

gRPC它是对RPC-种非常简洁的实现并且解决了很多RPC的问题。



# Protocol Buffers

https://developers.google.com/protocol-buffers/

可以用来定义消息和服务。然后，你只需要实现服务即可，剩余的gRPC代码将会自动生成。

.proto可以适用于十几种开发语言(包括服务端和客户端)，并且它允许你使用同一
个框架来支持每秒百万级以上的RPC调用。

gPRC使用的是合约优先的API开发模式，它默认使用Protocol buffers (protobuf) 作为接口设计语言(IDL)，这个.proto文件包括两部分:
* gRPC服务的定义
* 服务端和客户端之间传递的消息

好处
* 和开发语言无关
* 可以生成所有主流开发语言的代码
* 数据是二进制格式的，串行化的效率高，Payload比较小 
* 适合传递大量的数据
* 通过设定某些规则，是的API的进化也很简单

数据类型
* 数值型，数值型有很多种形式: double, float, int32, int64, uint32, uint64, sint32, sint64,fixed32, fixed64, sfixed32, sfixed64。
* 布尔型，bool型可以有True和False两个值。
* 字符串，string表示任意长度的文本，但是它必须包含的是UTF-8编码或7位ASCII的文本，长度不可超过232。
* 字节型，bytes可 表示任意的byte数组序列，但是长度也不可以超过232


字段的数值(Tag)
* 在Protocol Buffers里面，字段的名其实没那么重要，但是写C#代码的时候，字段名还是很重要的。
* 对于protobuf来说，tag更为重要。
* 可以使用的最小的tag数值是1，最大值是2^29- 1,或者536,870,911。不可用19000到19999之间的数，这部分数是保留的。
* 1到15的Tag数只占用1个字节的空间，所以它们应该被用在频繁使用的字段上。而从16到2047，则占用两个字节，它们可以用在不频繁使用的字段上。






















