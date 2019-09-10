---
layout:     post                    # 使用的布局（不需要改）
title:      报错记录	        # 标题 
subtitle:           #副标题
date:       2019-09-10              # 时间
author:     alice                      # 作者
catalog: true                       # 是否归档
tags:                               #标签
    - Redis
---



### 1.从github下载自己的项目，导入时发生错误

Cannot load module file 'C:/Users/42536/Desktop/QRcode-master/bootdo/bootdo.iml':FileC:\Users\42536\Desktop\xxQRcode-master\bootdo\bootdo.iml does not exist Would you like to remove module 'bootdo' from the project?

#### 解决

file---open recent----manage projects 列表中删除冲突项目

#### 参考

[链接](https://intellij-support.jetbrains.com/hc/en-us/community/posts/115000782030-Can-t-get-rid-of-deleted-module-Error-loading-project-Cannot-load-module-would-you-like-to-remove-from-project-)



2019.9.7

### 2.读取图片的绝对路径时，chrome报错

![](.\error_img\TIM图片20190908005209.png)

Not allowed to load local resource: file:///xxxxxxxxx，file表示本地资源；

页面调试时显示![](C:\Users\ZZX\Desktop\note\error_img\TIM图片20190908005336.jpg)这样的相对路径；

应该使用虚拟路径解决

#### 解决

写一个配置类设置虚拟路径

```java
@Component
public class MyWebAppConfiguration extends WebMvcConfigurerAdapter {
    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("/imgs/**").addResourceLocations("file:///c:/Users/42536/Desktop/xxQRcode-master/bootdo/src/main/resources/static/img/qrcode_img/");
    }
}
```

发现报错消失，然而图片仍然没有成功导入

 下断点后发现bootdo已经帮我配置好了一个虚拟路径，如果将图片放入/files/**对应的绝对路径图片可以成功显示出来。![](.\error_img\TIM图片20190908010548.png)

配置方法是相同的，全局搜索一下看看少写了什么条件

![](.\error_img\TIM图片20190908010842.png)

在shiroConfig.java中添加

```java
     filterChainDefinitionMap.put("/imgs/**", "anon");
```

这个类时shiro权限控制，“anon”表示任何人可以访问。

图片成功显示。



说了那么多，然后bc告诉我，你为什么不用fiddler？



### 3.无法访问此网站，xxx拒绝我们的请求

拒绝请求，服务器可ping但是端口不可连接。

#### 解决

80端口，通过看错误日志检查nginx运行情况。Log中没有具体错误，说明nginx异常。先打开了wdcp查看nginx的运行情况，重启未成功，遂在xshell中service nigixd restart，网页就成功显示了。

为什么会异常呢，可能是80端口与springboot发生了冲突。

于是在xx.html可打开的情况下，运行springboot，出现报错：

The Tomcat connector configured to listen on port 80 failed to start. The port may already be in use or the connector may be misconfigured.

Action:

Verify the connector's configuration, identify and stop any process that's listening on port 80, or configure this application to listen on another port.

方案如下：

1. 停止nginx，这个方案不太好

2. ①增加81端口给springboot ②wdcp→安全管理→防火墙，增加规则

③阿里云控制台→实例详情→本实例安全组→内网入增加安全组规则

80是http默认端口



### 4.MYSQL在linux环境下运行报错并且数据库多了些表名为大写的表

#### 解决

联想到或许是windows和linux下文件系统大小写敏感不同的问题，查找资料后，将my.conf中的lower_case_table_names=1修改成0，就消除了大小写敏感的问题.

#### 参考

[Linux下MySQL大小写敏感问题](https:// blog.csdn.net/zhaopeng_yu/article/details/80785813)



### 5.Div高度自适应屏幕

通过background属性想给div设置一个绿色，代码如图，然而没有效果

![](.\error_img\TIM图片20190910082816.png)

![](.\error_img\TIM图片20190910082836.png)

#### 解决

将代码修改如下：

```html
  *{
        margin: 0;
        padding: 0;
    }
    html, body{
        width: 100%;
        height: 100%;
    }

    .wechat_background{
        height:auto;
        width:100%;
        min-height: 100%;
        max-width: 100%;
        z-index:-1;
        position: absolute;
        left: 0;
        top: 0;
    }
```

Css中定义*{padding:0px;margin: 0px;}，*的意思为选择html中的所有元素。

Min-hight对元素的高度设置一个最低限制。元素可以比指定值高，但不能比其矮

![](C:\Users\ZZX\Desktop\note\error_img\TIM图片20190910083353.png)

参考

[div适应屏幕](https://blog.csdn.net/f120032777/article/details/83059893)