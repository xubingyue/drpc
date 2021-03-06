# DRPC

DRPC是一个C++ 智能RPC库,开发者只需要通过简单的接口定义服务名称以及服务实现函数，调用者直接使用参数进行调用，无需考虑封包和解包。

## 例子

使用本地进程内进行模拟测试（可以很简单的通过网络库发送请求即可跨机器使用此RPC库）
```
    RpcService<MsgpackProtocol> rpcServer;  // or RpcService<JsonProtocol> rpcServer
    RpcService<MsgpackProtocol> rpcClient;

    rpcServer.def("test", [](const std::string& value) {
        cout << "receive " << value << endl;
    });

    int reqID = 0;
    rpcServer.def("ping", [&reqID](RpcRequestInfo info, std::string& value) {
        cout << "receive " << value << endl;
        reqID = info.getRequestID();
    });

    static_assert(std::is_same<decltype(rpcServer), decltype(rpcClient)>::value, "");

    {
        auto requestBinary = rpcClient.call("test", "hello world?");
        rpcServer.handleRpc(requestBinary);
    }

    {
        auto requestBinary = rpcClient.call("ping", "hello world?", [](RpcRequestInfo info, std::string& value) {
            cout << "reply of " << value << endl;

        });
        rpcServer.handleRpc(requestBinary);


        auto replyBinary = rpcServer.reply(reqID, "hello world!");
        rpcClient.handleRpc(replyBinary);
    }

    {
        auto requestBinary = rpcClient.call("ping", "hello?", [](std::string& value) {
            cout << "reply of " << value << endl;

        });
        rpcServer.handleRpc(requestBinary);


        auto replyBinary = rpcServer.reply(reqID, "hello!");
        rpcClient.handleRpc(replyBinary);
    }
```