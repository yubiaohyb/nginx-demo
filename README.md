# niginx-demo
个人nginx学习
---
 * [windows](https://github.com/yubiaohyb/nginx-demo/blob/master/windows/windows.md) - windows版个人学习配置内容.
 * linux - linux版学习内容.
 
#### 注意事项 ####
```
  由于缓存问题可能导致看到的结果和预期存在差异！！！
```
# 官网文档阅读小计
## Beginner’s Guide
nginxg工作时包含一个主进程和若干工作进程。主进程读取使用配置文件，维护工作进程；工作进程则负责请求的实际处理。
nginx采用基于事件模型和操作系统依赖机制实现请求在工作进程间的高效分发。
工作进程的数量可以通过配置文件指定特定的数值，也可以自动与系统cpu内核数保持一致。

### Starting, Stopping, and Reloading Configuration
nginx -s signal
* stop — fast shutdown
* quit — graceful shutdown(This command should be executed under the same user that started nginx.)
* reload — reloading the configuration file
* reopen — reopening the log files

nginx重载配置(原理/过程)
>主进程接收到重载配置信号后，首先会校验新配置文件的语法正确性，然后尝试应用文件内配置。
>如果这一步执行成功，主进程会开启新的工作进程，并请求老的工作进程退出服务。
>老的工作进程在接收到退出请求后，会停止接收新的连接，继续完成已接收请求连接任务推出，直到全部完成再退出。
>否则，主进程会回滚变化，继续使用老的配置提供服务。

nginx.pid中保存的是主进程的进程id。
kill -s QUIT 1628同样可以实现nginx的优雅关闭，同理，应该可将QUIT替换成STOP实现nginx的快速关闭。

### Configuration File’s Structure
>nginx通过配置文件中的指令实现对模块的组织。
>指令分为简单指令和复杂指令。
>如果在一个块指令中包含其他的指令，则称该块指令为一个上下文（例如：events/http/server/location）。
>配置文件中，如果一个上下文不在其他上下文中，则认为该上下文处在主上下文中（如：events/http）。
>配置中使用#进行注释备注。

### Serving Static Content
略。

### Setting Up a Simple Proxy Server
略。

### Setting Up FastCGI Proxying
略。




