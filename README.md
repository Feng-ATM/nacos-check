# nacos
https://www.mubucm.com/doc/DINuBZd8aK   了解这个漏洞


Alibaba Nacos权限认证绕过漏洞复现
简介：
    Nacos（官方网站：http://nacos.io）是一个易于使用的平台，旨在用于动态服务发现，配置和服务管理。它可以帮助您轻松构建云本机应用程序和微服务平台。

 漏洞概述
    2020年12月29日，Nacos官方在github发布的issue中披露Alibaba Nacos 存在一个由于不当处理User-Agent导致的未授权访问漏洞 。通过该漏洞，攻击者可以进行任意操作，包括创建新用户并进行登录后操作。

影响版本
    Nacos <= 2.0.0-ALPHA.1

环境搭建
    Nacos下载地址(github):https://github.com/alibaba/nacos/releases/tag/2.0.0-ALPHA.1

【Linux搭建】
    下载好后解压文件：tar -zxvf nacos-server-2.0.0-ALPHA.1.tar.gz
    进入目录，执行搭建命令：./startup.sh -m standalone

【Windows搭建】
    解压进入目录后执行搭建命令：cmd startup.cmd -m standalone
    接着访问http://your-ip:8848/nacos
    默认账号密码nacos/nacos

出现Nacos登录页面则表示搭建成功
    漏洞复现
    访问：http://your-ip:8848/nacos/v1/auth/users?pageNo=1&pageSize=1
    可以查看到用户列表：
    从上图可以发现，目前有一个用户nacos
    漏洞利用，访问：http://your-ip:8848/nacos/v1/auth/users
    POST传参：usename=test1&password=test1
    修改UA头为Nacos-Server
    发送POST请求，返回码200，创建用户成功~！
    返回Nacos登录界面 test1/test1
    登录成功！
    或者直接用burp打，构造数据包poc如下：

漏洞分析
    首先，入口点我们看一下github上相关issues的讨论
    根据提出漏洞的大佬threedr3am描述，这个Nacos-Server是用来进行服务间的通信的白名单。比如服务A要访问服务B，如何知道服务A是服务，只需要在服务A访问服务B的时候UA上写成 Nacos-Server 即可。
    正因为这样，所以当我们UA恶意改为Nacos-Server的时候，就会被误以为是服务间的通信，因此在白名单当中，绕过的认证。
    这里用的是nacos-2.0.0-ALPHA.1的代码进行分析
    关键代码在该文件下：/nacos-2.0.0-ALPHA.1/naming/src/main/java/com/alibaba/nacos/naming/web/TrafficReviseFilter.java
    TrafficReviseFilter继承了Filter用来处理请求，而里面的doFilter的就很明确了。注释中写道，当接收到其他节点服务的请求时应该被通过，如何验证是其他服务。
    就是下面很简单的一个对于UA的一个判断逻辑
    这个Constants.NACOS_SERVER_HEADER跟踪一下，正是Nacos-Server
    经过这一层的验证，那么则进入到filterChain 过滤器链中的下一个filter过滤器，继续接下来的请求。

修复方式
    若业务环境允许，使用白名单限制相关web项目的访问来降低风险。
    官方已发布最新安全版本，请及时下载升级至安全版本。

