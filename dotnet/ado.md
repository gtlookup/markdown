# 3秒测试db连接

当db服务器连不上时，SqlConnection.Open 会等待30秒以上才能报异常。而 SqlConnection.ConnectionTimeout(默认是15秒) 又是只读的，后来在连接串中加上了 ==Connection Timeout=3==，debug一看 ConnectionTimeout 也确实是 3，但走到.Open()还是要等待30秒以上才报异常，没办法只能扒一段下面代码：

```c#
public static bool IsEnable(Func<bool> func)
{
    bool rlt = false; // 返回值

    Thread thread = new Thread(() => { rlt = func(); }); // 在线程里进程连接测试
    thread.IsBackground = true;
    Stopwatch sw = Stopwatch.StartNew(); // 计时开始（StartNew里调了Start()方法）
    thread.Start();

    TimeSpan timeout = TimeSpan.FromSeconds(3); // 3秒内连接不成功就返回
    // sw.Elapsed：从.Start()开始到现在的运行时间
    // thread.Join：线程阻塞一段时间
    while (!rlt && sw.Elapsed < timeout) thread.Join(TimeSpan.FromMilliseconds(200));
    sw.Stop(); // 结束

    return rlt;
}
```

```c#
// 测试sqlServer连接是否有效，同样也可以再写个方法来测试某个ip下的某个文件夹是否有效，只要重写一下线程里运行的代码就行
public static bool IsConnectionEnable()
{
    return IsEnable(() =>
    {
        SqlConnection cnn = null;
        try
        {
            cnn = new SqlConnection(Constant.INI_SQLSERVER_CONNECTION);
            cnn.Open();
            cnn.Close();
            return true;  // 连接有效
        }
        catch (SqlException e)
        {
            return false; // 连接无效
        }
        finally
        {
            if (cnn != null && cnn.State == ConnectionState.Open) cnn.Close();
        }
    });
}
```

