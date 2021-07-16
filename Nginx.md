### Nginx

**1.定义**

​		Nginx是一款轻量级的Web服务器、反向代理服务器，由于它的内存占用少，启动极快，高并发能力强，在互联网项目中广泛应用。使用C开发。官方称可以单节点可以支持5W并发连接数，实际生产环境可以承受2~3万并发。

**2.架构**

​		有一个`master`进程，读取并且验证nginx.conf+管理多个worker进程；

​		多个`worker`进程，在内部都会维护一个处理连接和请求的线程。

**3.热部署**

- [ ] master更新->提交worker更新

- [x] master更新->worker换代升级

4.优点

- 动静分离：将网站静态资源（网页HTML、img、CSS）与后台应用分开，提高用户访问静态资源的速度，降低对后台的访问。
- 反向代理：保障服务器的安全，实现负载均衡，实现IP访问控制。
- 健康检查：max_fials和failout_time

5.主要命令

`./nginx -s stop`+`./nginx -s quit`+`./nginx -s reload`







额外知识：

windows为读写文件提供了异步接口，Linux异步IO