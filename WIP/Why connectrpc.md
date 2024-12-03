用 protobuf 生成的 stub(connectts)  就可以得到大部分的收益

没有必要用grpc的encode/decode协议

简单

就是几条转换规则

不需要做改造 只需要一个 grpc 的 web proxy

![WXWorkCapture_17314796999129.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/c71ada4c-e76c-43a3-a285-f04c57387e58/92e3fc51-9ea4-4324-a104-763a6bd4c7c1/WXWorkCapture_17314796999129.png)

如上图 简单的对应关系 容易理解

使用 json 使得 浏览器的开发者工具 可以直接支持查看