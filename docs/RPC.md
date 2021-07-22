# RPC

RPC是一种通用性的系统通信手段，使得我们可以像调用本地方法一样调用远程系统提供的方法。

按照调用方式来看，RPC有四种模式：

- RR（Request-Response）模式，又叫请求响应模式，指每个调用都要有具体的返回结果信息。
- Oneway模式，又叫单向调用模式，调用即返回，没有响应的信息。
- Future模式，又叫异步模式，返回拿到一个Future对象，然后执行完获取到返回结果信息。
- Callback模式，又叫回调模式，处理完请求以后，将处理结果信息作为参数传递给回调函数进行处理。

这四种调用模式中，前两种最常见，后两种一般是RR和Oneway方式的包装，所以从本质上看，RPC一般对于客户端的来说是一种同步的远程服务调用技术。与其相对应的，一般来说MQ恰恰是一种异步的调用技术。



MQ（Message Queue，消息队列） 异步的远程调用，如果能同时存在很多个请求，该如何处理呢？进一步地，由于不能立即拿到处理结果，假若需要考虑失败策略，重试次数等，应该怎么设计呢？ 如果有N个不同系统相互之间都有RPC调用，这时候整个系统环境就是一个很大的网状结构，依赖关系有N*(N-1)/2个。