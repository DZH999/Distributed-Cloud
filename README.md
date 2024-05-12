# 文件服务器系统

本项目分为两个模块：**客户端**和**文件服务器**。客户端采用Qt框架实现，提供了用户友好的操作界面，支持**用户注册、登入、文件上传**和**下载**等功能。客户端与文件服务器之间是通过Qt的Http进行通信。文件服务器使用Nginx作为**Web服务器**，并借助**FastCG**I来处理客户端发来的**动态请求数据**。为了**避免单点故障**和**应对业务增长**，本项目采用了**fastDFS集群**，实现了**纵向备份**和**横向扩容**功能。本文件服务系统不仅适用于企业内部文件管理，还可广泛应用于**医院**、**电商**、**短视频**等领域。


项目亮点：
- 客户端通过使用单例模式来存储文件列表界面频繁使用的登入信息，避免频繁的创建销毁对象；
- 文件服务器系统端采用了Nginx和FastCGI 进程，实现了Web服务器动态请求数据的处理；
- 利用Fastdfs集群实现高性能、高可用文件上传和下载服务，同时还具备动态扩容功能；
- 使用Mysql数据库记录用户信息，并利用 Redis存储服务器频繁访问的数据。



# 技术栈

C++、Linux、Qt、Nginx、FastCGI、Fastdfs、Redis、Mysql

# 系统框架

![image](https://github.com/DZH999/Distributed-Cloud/assets/68979616/ac9f1603-47b3-47b2-9fb3-e3ebb6040355)


# 系统解析
- 该项目基于集群fastDFS分布式文件系统实现的。

**FastDFS介绍**

- 是由余庆用c语言编写的一款开源的分布式文件系统；
- 充分考虑**冗余备份**、**负载均衡**、**线性扩容**等机制，注重**高可用**、**高性能**等指标
  - 冗余备份: 纵向扩容
  - 线性扩容: 横向扩容

- fastDFS框架中的三个角色
  - 追踪器 ( Tracker ) - 管理者 - 守护进程 
    - 管理存储节点 
  - 存储节点 (storage)  - 守护进程 
    - 存储节点是有多个的 
  - 客户端 - 不是守护进程, 这是程序猿编写的程序
    -  文件上传 
    - 文件下载

- fastDFS三个角色之间的关系
  - 上传
![image-20240306153219080](https://github.com/DZH999/Distributed-Cloud/assets/68979616/bbc8906e-e51f-4b24-a0a7-be60eb1601b0)

  - 下载
![image-20240306153338337](https://github.com/DZH999/Distributed-Cloud/assets/68979616/d6e1d07a-2895-4e7e-829b-82ca0af1d8b8)


1、追踪器

- 最先启动追踪器

2、存储节点

- 第二启动
- 存储节点启动之后，会单独开一个线程
  - 汇报当前存储节点的容量和剩余容量；
  - 汇报数据同步情况；
  - 汇报数据被下载的情况。

3、客户端

- 最后启动
- 上传
  - 连接追踪器, 询问存储节点的信息;
    - 我要上传1G的文件, 询问那个存储节点有足够的容量 
  - 追踪器查询, 得到结果;
  - 追踪器将查到的存储节点的IP+端口发送给客户端;
  - 通过得到IP和端口连接存储节点;
  - 将文件内容发送给存储节点.
- 下载
  - 连接追踪器, 询问存储节点的信息;
    - 问一下, 要下载的文件在哪一个存储节点
  - 追踪器查询, 得到结果;
  - 追踪器将查到的存储节点的IP+端口发送给客户端;
  - 通过得到IP和端口连接存储节点;
  - 下载文件

4、fastDFS集群

![image-20240306154008255](https://github.com/DZH999/Distributed-Cloud/assets/68979616/090aa8cb-bff7-42b8-a8e0-c65e1e502b24)

1、追踪器集群

- 为什么集群？
  - 避免单点故障；
- 多个Tracker如何工作？
  - 轮询工作；
- 如何实现集群？
  - 修改配置文件；

2、存储节点集群

- fastDFS管理存储节点的方式？
  - 通过分组的方式完成；
- 集群方式（扩容方式）
  - 横向扩容--增加容量
    - 添加一台主机
    - 假设当前有2个组：group1，group2
      - 添加一个新组group3     -->新主机属于第三组
    - 不同主机不需要通信；
  - 纵向扩容--数据备份
    - 假设当前有两个组: group1, group2 
      - 将新的主机放到现有的组中 
      - 每个组的主机数量从1 -> N 
        - 这n台主机的关系就是相互备份的关系 
        - 同一个组中的主机需要通信 
        - 每组的容量 == 容量最小的这台主机
- 如何实现？
  - 修改配置文件
 

- 该项目使用Nginx和FastCGI作为web服务器，使用Nginx作为反向代理服务器。
  - 反向代理服务器可以**将客户端的请求分发到多个后端服务器上**，实现**负载均衡**，从而提高系统的性能和可靠性。并且反向代理服务器还可以**缓存后端服务器返回的资源**，如静态文件、网页内容等，从而**减轻后端服务器的负载**，加快客户端访问速度。
Nginx的优势：
- 更快
  - 高峰期(数以万计的并发时)nginx可以比其它web服务器更快的响应请求；
- 高扩展
  - 低耦合设计的模块组成，丰富第三方模块支持；
- 高可靠
  - 经过大批网站检验     新浪、迅雷、163
  - 每个worker进程相对独立, 出错之后可以快速开启新的worker
- 低内存消耗
  - 一般情况下,10000个非活跃的HTTP Keep-Alive连接在nginx中仅消耗 2.5M内存

Nginx配置

- Nginx配置文件的组织格式

![image-20240306100434414](https://github.com/DZH999/Distributed-Cloud/assets/68979616/a43e9dc3-1584-4bbf-9b12-a150f5ee127f)

- http模块->http相关的通信设置
  - server模块->每个server对应的是一台web服务器
    - location模块 ->处理的是客户端的请求
      - 例如：浏览器请求： http://192.168.10.100:80/login.html
      - 服务器处理客户端的请求       -------服务器要处理的指令如何从url中提取?
        - 去掉协议；
        - 去掉IP/域名+端口：192.168.10.100:80
        - 最后如果是文件名，去掉该名字：login.html
        - 剩下：/
        - 最终，服务器要处理的location指令:  `location / { 处理动作  }`
- mail模块->处理邮件相关动作

Nginx 部署静态网页

```c
 # 对应这个请求服务器要添加一个location
 location /
 {
    # 找一个静态网页
    root yundisk;  # 相对于/usr/local/nginx/来找
    # 客户端的请求是一个目录, nginx需要找一默认显示的网页
    index index.html index.htm;
 }
 # 配置之后重启nginx
 sudo nginx -s reload
```

均衡负载

![image-20240306141425133](https://github.com/DZH999/Distributed-Cloud/assets/68979616/8e2e7c04-b38d-4959-afaf-d5dee40bb499)



Nginx 部署动态网页

- 客户端会将数据提交给服务器

```c++
 # 使用get方式提交数据得到的url
 http://localhost/login?user=zhang3&passwd=123456&age=12&sex=man
    - http: 协议
    - localhost: 域名
    - /login: 服务器端要处理的指令
    - ? : 连接符, 后边的内容是客户端给服务器提交的数据
    - & : 分隔符
动态的url如何找服务器端处理的指令?
        - 去掉协议
        - 去掉域名/IP
        - 去掉端口
        - 去掉?和它后边的内容
```

1、请求消息(Request) - 客户端(浏览器)发送给服务器的数据格式

> 四部分: 请求行, 请求头, 空行, 请求数据 
>
> - 请求行: 说明请求类型, 要访问的资源, 以及使用的http版本 
>
> - 请求头: 说明服务器要使用的附加信息 
>
> - 空行: 空行是必须要有的, 即使没有请求数据 
>
> - 请求数据: 也叫主体, 可以添加任意的其他数据

- Get方式提交数据

> 第1行: 请求行 
>
> 第2-9行: 请求头(键值对) 
>
> 第10行: 空行 
>
> get方式提交数据, 没有第四部分, 提交的数据在请求行的第二部分, 提交的数据会全部显示在地址栏中

- Post方式提交数据 

> 第1行: 请求行
>
> 第2 -12行: 请求头 (键值对) 
>
> 第13行: 空行 
>
> 第14行: 提交的数据

2、响应消息(Response) -> 服务器给客户端发送的数据

> 四部分: 状态行, 消息报头, 空行, 响应正文 
>
> - 状态行: 包括http协议版本号, 状态码, 状态信息 
>
> - 消息报头: 说明客户端要使用的一些附加信息 
>
> - 空行: 空行是必须要有的 
>
> - 响应正文: 服务器返回给客户端的文本信息
>
> 第1行:状态行 
>
> 第2 -11行: 响应头(消息报头) 
>
> 第12行: 空行 
>
> 第13-18行: 服务器给客户端回复的数据



3、http状态码

> 状态代码有三位数字组成，第一个数字定义了响应的类别，共分五种类别: 
>
> - 1xx：指示信息--表示请求已接收，继续处理 
> - 2xx：成功--表示请求已被成功接收、理解、接受 
> - 3xx：重定向--要完成请求必须进行更进一步的操作
> - 4xx：客户端错误--请求有语法错误或请求无法实现 
> - 5xx：服务器端错误--服务器未能实现合法的请求



**CGI**

> **通用网关接口（Common Gateway Interface，CGI）**描述了**客户端**和**服务器程序**之间传输数据的一种标准，可以让**一个客户端从网页浏览器向执行在网络服务器上的程序请求数据**。CGI 独立于任何语言的，CGI 程序可以用任何脚本语言或者是完全独立编程语言实现，只要这个语言可以在这个系统上运行。
>
> http://localhost/login?user=zhang3&passwd=123456&age=12&sex=man
>
> 1. 用户通过浏览器访问服务器, 发送了一个请求, 请求的url如上
> 2. 服务器接收数据, 对接收的数据进行解析 
> 3. nginx对于一些登录数据不知道如何处理, nginx将数据发送给了cgi程序 
>    - 服务器端会创建一个cgi进程
> 4.  CGI进程执行 
>    - 加载配置, 如果有需求加载配置文件获取数据 ；
>    - 连接其他服务器: 比如数据库； 
>    - 逻辑处理； 
>    - 得到结果, 将结果发送给服务器 ；
>    - 退出 。
> 5. 服务器将cgi处理结果发送给客户端 在服务器端CGI进程会被频繁的创建销毁 
>    - 服务器开销大, 效率低

**fastCGI**

> 快速通用网关接口（Fast Common Gateway Interface／FastCGI）是**通用网关接口**（CGI）的改进，描述了客户端 和服务器程序之间传输数据的一种标准。 **FastCGI致力于减少Web服务器与 CGI 程序 之间互动的开销，从而使服务器可以同时处理更多的Web请求** 。与为每个请求创建一个新的进程不同，FastCGI使用持续的进程来处理 一连串的请求。这些进程由**FastCGI进程管理器**管理，而不是web服务器。

fastCGI与CGI的区别：CGI 就是所谓的短生存期应用程序，FastCGI 就是所谓的长生存期应用程序。FastCGI像是一个常驻(long-live)型的 CGI，它可以一直执行着，不会每次都要花费时间去fork一次。

**使用spawn-fcgi作为FastCGI进程管理器**

> spawn-fcgi是一个通用的**FastCGI进程管理器**，简单小巧，原先是属于lighttpd的一部分，后来由于使用比较广 泛，所以就迁移出来作为独立项目了。spawn-fcgi使用pre-fork 模型，功能主要是**打开监听端口，绑定地址，然 后fork-and-exec创建我们编写的fastcgi应用程序进程，退出完成工作**。fastcgi应用程序初始化，然后进入死循环侦听socket的连接请求。


- 该项目使用的数据库是Mysql和Redis。所用的数据存储在Mysql中，但客户端访问服务器时，有一些数据，服务器需要频繁的查询数据。
  - 服务器第一次访问某些数据，先从Mysql中读出，并将数据写入Redis中；
  - 服务器第二次直接从redis中读数据。


# 补充知识

**单例模式**

1、单例模式的优点：

- 在内存中只有一个对象，节省内存空间；
- 避免频繁的创建销毁对象，可以提高性能；
- 避免对共享资源的多重占用；
- 可以全局访问。

2、单例模式的适用场景：

- 需要频繁实例化然后销毁对象
- 创建对象耗时过多或者耗资源过多，但又经常使用对象。
- 有状态的工具类对象
- 频繁访问数据库或文件的对象
- 要求只有一个对象的场景

3、如何保证单例模式对象只有一个？

```c++
 // 在类外部不允许进行new操作
class Test
 {
 public:
    // 1. 默认构造
    // 2. 默认析构
    // 3. 默认的拷贝构造
// 1. 构造函数私有化
// 2. 拷贝构造私有化
}
```

4、单例模式实现方式？

- 懒汉模式-- 单例对象在使用的时候被创建出来，线程安全问题需要考虑

  ```c++
  class Singleton {
  private:
      static Singleton* instance;
      Singleton() {} // 构造函数私有化，防止外部创建对象
  
  public:
      static Singleton* getInstance() {
          if (instance == nullptr) {
              instance = new Singleton();
          }
          return instance;
      }
  };
  
  Singleton* Singleton::instance = nullptr; // 静态成员变量初始化
  
  // 使用示例
  Singleton* obj = Singleton::getInstance();
  ```

- 饿汉模式-- 单例对象在使用之前被创建出来

```c++
#include <iostream>

class Singleton {
private:
    // 私有化构造函数和析构函数，防止外部创建和销毁对象
    Singleton() {}
    ~Singleton() {}
public:
    // 静态成员函数，返回类的唯一实例
    static Singleton* getInstance() {
        static Singleton instance; // 在静态函数中创建静态局部变量，保证对象只会被创建一次
        return &instance;
    }
};
// 使用示例
int main() {
    Singleton* obj1 = Singleton::getInstance();
    Singleton* obj2 = Singleton::getInstance();
    std::cout << "obj1 address: " << obj1 << std::endl;
    std::cout << "obj2 address: " << obj2 << std::endl;
    return 0;
}
```


# 效果演示
- 登入

![image](https://github.com/DZH999/Distributed-Cloud/assets/68979616/b9de89e9-f4ba-499f-894f-359cfc766f88)

- 注册

![image](https://github.com/DZH999/Distributed-Cloud/assets/68979616/c9bd3137-b1c2-4c4a-8187-efec99ae6dfe)


- 首页

![image](https://github.com/DZH999/Distributed-Cloud/assets/68979616/f1b119b4-0f5b-45d8-98e4-0c6dbcee579e)


- 上传

![image](https://github.com/DZH999/Distributed-Cloud/assets/68979616/0a912e2d-a055-4251-b30d-739454e02199)
![image](https://github.com/DZH999/Distributed-Cloud/assets/68979616/a11ca262-44b9-4b09-aa92-ba1a66ae9511)

- 下载

![image](https://github.com/DZH999/Distributed-Cloud/assets/68979616/c776e7f8-cfc9-4217-8a6f-bf782a4bfae5)

- 传输记录

![image](https://github.com/DZH999/Distributed-Cloud/assets/68979616/52b530e0-9a3f-4085-a80d-5333dadb272c)

- 等等

