# 修改.cshtml不用重启

```c#
public class Startup {
    public void ConfigureServices(IServiceCollection services)
    {
        // 如果红的，就通过nuget添加
        // Microsoft.AspNetCore.Mvc.Razor.RuntimeCompilation（注意要选和当前.net core版本相同的）
        services.AddRazorPages().AddRazorRuntimeCompilation();
    }
}
```

# 开启controller/view

```c#
public class Startup {
    public void ConfigureServices(IServiceCollection services)
    {
        services.AddControllersWithViews(); // 第一步
    }

    public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
    {
        if (env.IsDevelopment())
        {
            app.UseDeveloperExceptionPage();
        }

        app.UseRouting(); // 第二步

        app.UseEndpoints(endpoints => {
            // 第三步
            endpoints.MapControllerRoute(name: "default", pattern: "{controller=Home}/{action=Index}/{id?}");
            // 第四步：
            // 		添加Controllers目录，在里面添加HomeController->Index->return View()
            //      添加Views目录，在里面添加Home/Index.cshtml
        });
    }
}
```

# 引入js

```C#
app.UseStaticFiles(); // Startup.Configure(IApplicationBuilder app, IWebHostEnvironment env)
```

```html
<!-- wwwroot/lib/jquery/jquery.min.js -->
<script src="@Url.Content("~/lib/jquery/jquery.min.js")"></script>
```

