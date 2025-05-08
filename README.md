- CTP 6.3.19 win x68的cli/c++封装，基于.Net8
- 优化了项目结构，所有接口封装+用户自定义接口处理
- dll的显式指定，dll接口文件下载链接：https://www.simnow.com.cn/DocumentDown/api_3/5_2_2/traderapi_v6.3.19_P1.zip
- 原汁原味的数据结构名称
- xml文档生成

market:
```csharp
using Ctp.Net;

int reqId = 0;
MarketApi api = MarketApi.CreateApi("thostmduserapi_se.dll", "tmp/md_", false, false);
api.OnFrontConnected += Api_OnFrontConnected;
api.OnFrontDisconnected += Api_OnFrontDisconnected;
api.OnRspUserLogin += Api_OnRspUserLogin;
api.OnRtnDepthMarketData += Api_OnRtnDepthMarketData;
api.OnRspError += Api_OnRspErrorAsync;
api.RegisterFront($"tcp://{market_ip}:{market_port}");
api.Init();
await Task.Delay(Timeout.Infinite);

private void Api_OnFrontConnected()
{
    api.ReqUserLogin(new(), reqId++);
}

private void Api_OnFrontDisconnected(int reason)
{
    Console.WriteLine($"[market] 已断开");
}

private void Api_OnRspUserLogin(RspUserLoginField rspUserLogin, RspInfoField rspInfo, int requestID, bool isLast)
{
    if ((rspInfo?.ErrorID ?? 0) == 0 && isLast)
        Console.WriteLine($"[market] 已登录");
    else
        Console.WriteLine($"[market] {rspInfo?.ErrorID} {rspInfo?.ErrorMsg}");
}

private void Api_OnRspErrorAsync(RspInfoField rspInfo, int requestID, bool isLast)
{
    Console.WriteLine($"[market] {rspInfo.ErrorMsg}");
}

private void Api_OnRtnDepthMarketData(DepthMarketDataField depthMarketData)
{
}
```

trader:
```csharp
using Ctp.Net;

int reqId = 0;
TraderApi api = TraderApi.CreateApi("thosttraderapi_se.dll", "tmp/md_");
api.OnFrontConnected += Api_OnFrontConnected;
api.OnFrontDisconnected += Api_OnFrontDisconnected;
api.OnRspAuthenticate += Api_OnRspAuthenticate;
api.OnRspUserLogin += Api_OnRspUserLogin;
api.OnRspQryTradingAccount += Api_OnRspQryTradingAccount;
api.OnRspQryInvestorPosition += Api_OnRspQryInvestorPosition;
api.OnRspQryOrder += Api_OnRspQryOrder;
api.OnRtnOrder += Api_OnRtnOrder;
api.OnRspError += Api_OnRspError;
api.SubscribePrivateTopic(RESUME_TYPE.QUICK);
api.RegisterFront($"tcp://{trader_ip}:{trader_port}");
api.Init();
await Task.Delay(Timeout.Infinite);

private void Api_OnFrontConnected()
{
    Console.WriteLine($"已连接");

    var auth = new ReqAuthenticateField()
    {
        AppID = "",
        AuthCode = "",
        BrokerID = "",
        UserID = "",
    };
    api.ReqAuthenticate(auth, _reqId++);
}

private void Api_OnFrontDisconnected(int reason)
{
    Console.WriteLine($"已断开");
}

private void Api_OnRspAuthenticate(RspAuthenticateField rspAuthenticateField, RspInfoField rspInfo, int requestID, bool isLast)
{
    if (rspInfo?.ErrorID > 0)
    {
        Console.WriteLine($"认证失败{rspInfo?.ErrorID}{rspInfo?.ErrorMsg}");
        return;
    }
    var req = new ReqUserLoginField()
    {
        BrokerID = "",
        UserID = "",
        Password = "",
    };
    api.ReqUserLogin(req, _reqId++);
    Console.WriteLine($"已认证");
}

private void Api_OnRspUserLogin(RspUserLoginField rspUserLogin, RspInfoField rspInfo, int requestID, bool isLast)
{
    if (rspInfo?.ErrorID > 0)
    {
        Console.WriteLine($"登录失败{rspInfo?.ErrorID}{rspInfo?.ErrorMsg}");
        return;
    }
    Console.WriteLine($"已登录");
}

private void Api_OnRspQryInvestorPosition(InvestorPositionField investorPosition, RspInfoField rspInfo, int requestID, bool isLast)
{
    if (rspInfo?.ErrorID > 0)
    {
        Console.WriteLine($"{nameof(_api.ReqQryInvestorPosition)} {rspInfo?.ErrorID}{rspInfo?.ErrorMsg}");
        return;
    }
}

private void Api_OnRspQryTradingAccount(TradingAccountField tradingAccount, RspInfoField rspInfo, int requestID, bool isLast)
{
    if (rspInfo?.ErrorID > 0)
    {
        Console.WriteLine($"{nameof(_api.ReqQryTradingAccount)} {rspInfo?.ErrorID}{rspInfo?.ErrorMsg}");
        return;
    }
}

private void Api_OnRspQryOrder(OrderField order, RspInfoField rspInfo, int requestID, bool isLast)
{
    if (rspInfo?.ErrorID > 0)
    {
        Console.WriteLine($"{nameof(_api.ReqQryOrder)} {rspInfo.ErrorID} {rspInfo.ErrorMsg}");
        return;
    }
}

private void Api_OnRtnOrder(OrderField order)
{
}

private void Api_OnRspError(RspInfoField rspInfo, int requestID, bool isLast)
{
    Console.WriteLine($"{nameof(_api.OnRspError)} {rspInfo.ErrorID} {rspInfo.ErrorMsg}");
}
```