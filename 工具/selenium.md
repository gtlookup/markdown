# 依赖库

```xml
<dependency>
    <groupId>org.seleniumhq.selenium</groupId>
    <artifactId>selenium-java</artifactId>
    <version>3.141.59</version>
</dependency>
```

# 下载浏览器驱动

http://npm.taobao.org/mirrors/chromedriver

# 示例

```java
public class Demo1 {
    public static void main(String[] args) throws InterruptedException {
        // 导入下载的浏览驱动
        System.setProperty("webdriver.chrome.driver", "C:\\Users\\uesr\\Downloads\\chromedriver.exe");
        // 设置浏览器后台运行
        //ChromeOptions ops = new ChromeOptions();
        //ops.addArguments("--headless");
        // 启动浏览器
        //ChromeDriver dr = new ChromeDriver(ChromeDriverService.createDefaultService(), ops);
        ChromeDriver dr = new ChromeDriver();

        // 最大化窗口
        dr.manage().window().maximize();
        // 与浏览器同步非常重要，必须等待浏览器加载完毕
        dr.manage().timeouts().implicitlyWait(10, TimeUnit.SECONDS);
        // 打开网站
        dr.get("http://101.133.139.165/");
        TimeUnit.SECONDS.sleep(2);

        // 执行 javascript
//        String script = "document.body.innerHTML += \"<div id='dv1' style='z-index:10000;position:absolute;width:100vw;height:100vh;background-color:#FF0'></div>\"";//"window.onload = function() {document.body.innerHTML = \"<div style='width:100vw;height:100vh;background-color:#FF0'></div>\"}";
//        dr.executeScript(script);
        
        // 设置账号
        dr.findElementById("txtUserID").sendKeys("healthcare");
        // 设置密码
        dr.findElementById("txtPassWord").sendKeys("healthcare");
        // 点按钮后又弹出个窗口
        dr.findElementById("login").click();
        // 点弹出窗口里的按钮后才 submit 登陆
        dr.findElementById("dodOK").click();

        Scanner input = new Scanner(System.in);
        input.next();
        input.close();
        // 关闭
        dr.quit();
    }
}
```

