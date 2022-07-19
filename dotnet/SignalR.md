# 跑起来

```c#
public class Startup {
    public void ConfigureServices(IServiceCollection services)
    {
        services.AddControllersWithViews();
        services.AddRazorPages().AddRazorRuntimeCompilation();
        services.AddSignalR(); // 添加SignalR服务
    }

    public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
    {
        if (env.IsDevelopment())
        {
            app.UseDeveloperExceptionPage();
        }

        app.UseStaticFiles(); // 开启页面可引入js/css功能
        app.UseRouting();

        app.UseEndpoints(endpoints => {
            endpoints.MapHub<ChatHub>("/chathub"); // 服务类及url
            endpoints.MapControllerRoute(name: "default", pattern: "{controller=Home}/{action=Index}/{id?}");
        });
    }
}
```

```c#
public class ChatHub : Hub {
    public async Task SendMessage(string v1, string v2, string v3) { // 给客户端调的方法
        await Clients.Others.SendAsync("ReceiveMessage", arg1, arg2, arg3); // 群发给其它人
        await Clients.Caller.SendAsync("SendSuccess", "发送成功"); // 告诉自己，消息发送成功
    }
}
```

```html
<!-- 客户端 -->
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
    <script src="@Url.Content("~/lib/jquery/jquery.min.js")"></script>
    <!-- npm init -y -->
    <!-- npm install @microsoft/signalr -->
    <script src="@Url.Content("~/lib/signalr/signalr.js")"></script> <!-- signalr.js通过上面npm得来的 -->
    <script>
    $(function () {
        window.OB = {
            cnn: null,
            init: function () {
                OB.cnn = new signalR.HubConnectionBuilder().withUrl("/chatHub").build(); // new 个连接
                $('#btnSend').click(OB.onSend); // 发送事件
                OB.cnn.on('ReceiveMessage', OB.onReceive); // 接收事件
                OB.cnn.on('SendSuccess', OB.onSendOk); // 发送完回调事件
                OB.cnn.start().then(OB.onConnected).catch(OB.onDisconnected); // 发起连接，成功then，失败catch
            },
            onReceive: function (v1, v2, v3) {
                alert(v1 + v2 + v3);
            },
            onSendOk: function (msg) {
                alert(msg);
            },
            onSend: function () { // 调服务端 SendMessage 方法（似多少参数都行，看方法怎么定义）
                OB.cnn
                    .invoke('SendMessage', $('#txtV1').val(), $('#txtV2').val(), $('#txtV3').val())
                    .catch(OB.onDisconnected); // 异常
            },
            onConnected: function () {},
            onDisconnected: function (err) {
                console.error(err);
            }
        };
        OB.init();
    });
    </script>
</head>
<body>
<input id="txtV1" type="text" />
<input id="txtV2" type="text"/>
<input id="txtV3" type="text"/>
<button id="btnSend" type="button">send</button>
</body>
</html>
```

# 连接时传参

```javascript
OB.cnn = new signalR.HubConnectionBuilder().withUrl("/chatHub?user=aaa").build();
```

```c#
string user = Context.GetHttpContext().Request.Query["user"]; // 结果：aaa
```

