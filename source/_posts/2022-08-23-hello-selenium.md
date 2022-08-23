---
title: hello,selenium
date: 2022-08-23 19:03:48
tags:

 - tech
 - java
 - web

---



# Selenium

> Selenium automates browsers. That's it!

这个解释简单明了,就是用代码去操控浏览器. API简洁易用,想象空间很大.



# Java example

### Demo场景

1. 打开一个网站
2. 使用用户名密码登录
3. 屏幕截图并保存
4. 关闭浏览器

### 前置条件

安装好浏览器,基本主流浏览器都可以,后面会用chrome举例

###  具体步骤

1. 添加selenium的maven依赖

```xml
<!-- selenium -->
<dependency>
  <groupId>org.seleniumhq.selenium</groupId>
  <artifactId>selenium-java</artifactId>
  <version>4.4.0</version>
</dependency>

<!-- webdrivermanager -->
<dependency>
  <groupId>io.github.bonigarcia</groupId>
  <artifactId>webdrivermanager</artifactId>
  <version>5.2.3</version>
</dependency>

<!-- webdrivermanager非必须,也可以手动根据浏览器版本下载 -->
<!-- 基本每个浏览器都用对应的官方维护的driver -->
<!-- https://www.selenium.dev/documentation/webdriver/getting_started/install_drivers/ -->
```

2. java code

```java
package chrometest.service;

import java.io.File;

import org.apache.commons.io.FileUtils;
import org.openqa.selenium.By;
import org.openqa.selenium.Dimension;
import org.openqa.selenium.OutputType;
import org.openqa.selenium.TakesScreenshot;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.chrome.ChromeDriver;
import org.openqa.selenium.chrome.ChromeOptions;

import io.github.bonigarcia.wdm.WebDriverManager;

public class HelloSeleniumTest {

    public static void main(String[] args) throws Exception {

        // 也可以手动指定driver
        // System.setProperty("webdriver.chrome.driver", "/usr/bin/chromedriver");

        WebDriverManager.chromedriver().setup();

        ChromeOptions chromeOptions = new ChromeOptions();
        // hide window
        chromeOptions.setHeadless(true);
        // 也可以手动指定浏览器的可执行文件
        // chromeOptions.setBinary("/usr/bin/google-chrome");

        WebDriver driver = new ChromeDriver(chromeOptions);

        driver.get("https://chengchao.name/springrun/welcome/welcome.html");

        // resize window's size
        Dimension dem = new Dimension(1400, 1000);
        driver.manage().window().setSize(dem);

        // login
        WebElement usernameElement = driver.findElement(By.id("usernameInput"));
        usernameElement.sendKeys("demo-selenium-user");
        driver.findElement(By.id("passwordInput")).sendKeys("selenium_********");
        Thread.sleep(2000);
        driver.findElement(By.className("btn-block")).click();

        // TakesScreenshot
        File srcFile = ((TakesScreenshot)driver).getScreenshotAs(OutputType.FILE);
        String userhome = System.getProperty("user.home");
        FileUtils.copyFile(srcFile, new File(userhome + "/screenshot.png"));

        Thread.sleep(5000);
        driver.quit();
    }
}

```



### Linux 环境

可以使用docker来测试

```
docker pull ubuntu:20.04
docker run -itd --name myubuntu -v /Users/charles/Desktop/mydata:/mydata ubuntu:20.04
```

这里比较麻烦的是安装chrome浏览器,如果是arm平台,可以直接firefox,也是一样的

```shell
# chrome,暂时不支持arm架构
# 1. import the GPG key
wget -q -O - https://dl.google.com/linux/linux_signing_key.pub | sudo apt-key add -
# 2. add source list
sh -c 'echo "deb [arch=amd64] http://dl.google.com/linux/chrome/deb/ stable main" >> /etc/apt/sources.list.d/google-chrome.list'
# 3. update&install
apt update
apt install google-chrome-stable -y

# firefox
apt install firefox

# 安装中文字体
apt install ttf-wqy-microhei ttf-wqy-zenhei xfonts-intl-chinese
```

```java
// 如果是firefor,简单修改一下代码就可以了,别的都不需要改
System.setProperty("webdriver.gecko.driver", "/root/geckodriver");
FirefoxOptions firefoxOptions = new FirefoxOptions();
firefoxOptions.setBinary("/usr/bin/firefox");
firefoxOptions.setHeadless(true);
WebDriver driver = new FirefoxDriver(firefoxOptions);
```



