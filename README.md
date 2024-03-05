# 分布式云盘系统

项目提供文件存储、同步、访问（上传、下载）等服务，解决了大容量存储和负载均衡的问题。项目在Linux系统上进行部署，并通过QT来呈现用户界面，完成各项功能。
项目难点：
- 基于C/S网络架构和QT搭建的客户端，包含登入、注册、上传下载文件等功能；
- Web服务器是基于Nginx和fastCGI实现的；
- 基于Fastdfs的文件上传、下载服务；
- 基于Redis和Mysql存储数据。


# 技术栈

Linux、C++、QT、Nginx、Redis、Mysql、Fastdfs

# 系统框架

![image](https://github.com/DZH999/Distributed-Cloud/assets/68979616/ac9f1603-47b3-47b2-9fb3-e3ebb6040355)


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

# 系统解析
