# 休眠/恢复事件

```c#
public NFAgreeMain()
{
    SystemEvents.PowerModeChanged += (sender, e) =>
    {
        if (e.Mode == PowerModes.Resume) // 恢复
        {
            // 重点注意：之前的写db类是静态单例的，导致写db偶尔好用，大部分情况不好用
            //         给改回用的时候new，就次次好用了
            new DComputerLogMapper<SqlLiteDB>().Insert("06");
        }
        
        if (e.Mode == PowerModes.Suspend) // 休眠
        {
            new DComputerLogMapper<SqlLiteDB>().Insert("05");
        }
    };
    InitializeComponent();
}
```

# 关机/重启事件

```c#
public NFAgreeMain()
{
    SystemEvents.SessionEnding += (sender, e) =>
    {
        // 注意点和上面一样
        new DComputerLogMapper<SqlLiteDB>().Insert("02");
    };
    InitializeComponent();
}
```

# 进程操作

```c#
Process[] ps = Process.GetProcesses();                // 获取全部运行的进程
Process ps = Process.GetCurrentProcess();             // 获取当前运行的.net程序进程
Process[] ps = Process.GetProcessesByName("notepad"); // 获取记事本进程
Process.GetProcessesByName("notepad")[0].Kill();      // 关闭记事本进程
```

# 允许Console.WriteLine

```c#
[DllImport("kernel32.dll")]
private static extern bool AllocConsole();
public NFAgreeMain()
{
    AllocConsole();
    InitializeComponent();
}
```

# 管道通信

```c#
public class NamedPipe
{
    public static void Send(string msg) // 发送端
    {
        new Thread(o =>
        {
            using (NamedPipeClientStream pipe = new NamedPipeClientStream("localhost", "9527", PipeDirection.InOut, PipeOptions.None, TokenImpersonationLevel.None))
            {
                pipe.Connect(); // 连接接收端
                // using {} 感觉有点儿像 java 的 synchronized
                // 这里以及下面不加using会报错
                using (StreamWriter sw = new StreamWriter(pipe))
                {
                    sw.WriteLine(msg);
                    sw.Flush();
                }   
            }
        }).Start();
    }

    public static void Close() // 发关关闭消息，防止窗口关闭后进程还存在
    {
        Send("close");
    }

    public static void Receive() // 接收端
    {
        new Thread(o =>
        {
            while (true)
            {
                using (NamedPipeServerStream pipe = new NamedPipeServerStream("9527", PipeDirection.InOut, 1, PipeTransmissionMode.Byte))
                {
                    pipe.WaitForConnection(); // 等待发送端连接
                    using (StreamReader sw = new StreamReader(pipe))
                    {
                        string s = sw.ReadToEnd();
                        if (s.StartsWith("close")) break; // 如果是关闭消息就把无限循环结束，否则窗口关闭后进程还会存在
                        Console.WriteLine(s);
                        
                    }
                }
            }
        }).Start();
    }
}
```



