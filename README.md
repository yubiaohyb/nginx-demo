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

## Admin’s Guide
略。

## Controlling nginx
使用信号可以控制nginx。
主进程的进程id默认写在/usr/local/nginx/logs/nginx.pid文件中。
名称可以在配置期改变或者在nginx.conf中通过pid指令指定。
主进程支持以下信号：
```
TERM, INT	fast shutdown
QUIT	 graceful shutdown
HUP	  changing configuration, keeping up with a changed time zone (only for FreeBSD and Linux), starting new worker processes with a new       configuration, graceful shutdown of old worker processes
USR1	 re-opening log files
USR2	 upgrading an executable file
WINCH	graceful shutdown of worker processes
```
工作进程支持以下信号：
```

TERM, INT	fast shutdown
QUIT	graceful shutdown
USR1	re-opening log files
WINCH	abnormal termination for debugging (requires debug_points to be enabled)
```
### Changing Configuration
向主进程发送HUP信号，nginx会重新读取配置。
主进程首先会检查语法正确性，然后尝试应用新配置，也就是说要打开log文件和新的监听socket。
如果失败，会回滚变动，继续使用老的配置工作。
如果成功，则会创建新的进程，并向老的进程发送消息请求优雅关闭。老的进程会关闭监听socket，完成已有的终端服务连接，在所有请求任务完成后关闭。
假设nginx运行在FreeBSD环境中，使用命令 ps axw -o pid,ppid,user,%cpu,vsz,wchan,command | egrep '(nginx|PID)' 可以查看当前nginx相关进程。
正常如下：
```
  PID  PPID USER    %CPU   VSZ WCHAN  COMMAND
33126     1 root     0.0  1148 pause  nginx: master process /usr/local/nginx/sbin/nginx
33127 33126 nobody   0.0  1380 kqread nginx: worker process (nginx)
33128 33126 nobody   0.0  1364 kqread nginx: worker process (nginx)
33129 33126 nobody   0.0  1364 kqread nginx: worker process (nginx)
```
HUP信号发送后，如下：
```
  PID  PPID USER    %CPU   VSZ WCHAN  COMMAND
33126     1 root     0.0  1164 pause  nginx: master process /usr/local/nginx/sbin/nginx
33129 33126 nobody   0.0  1380 kqread nginx: worker process is shutting down (nginx)
33134 33126 nobody   0.0  1368 kqread nginx: worker process (nginx)
33135 33126 nobody   0.0  1368 kqread nginx: worker process (nginx)
33136 33126 nobody   0.0  1368 kqread nginx: worker process (nginx)
```
老的PID为33129的工作进程仍然工作。一段时间后，再次查看：
```
  PID  PPID USER    %CPU   VSZ WCHAN  COMMAND
33126     1 root     0.0  1164 pause  nginx: master process /usr/local/nginx/sbin/nginx
33134 33126 nobody   0.0  1368 kqread nginx: worker process (nginx)
33135 33126 nobody   0.0  1368 kqread nginx: worker process (nginx)
33136 33126 nobody   0.0  1368 kqread nginx: worker process (nginx)
```
### Rotating Log-files
分割日志文件前，首先需要对文件进行重命名。然后会向主进程发送USR1信号。
主进程作为持有者，会重新打开所有当前已打开的日志文件，然后将它们的权限置为nobody，所有的工作进程也是工作在这个权限下的。
在重打开操作成功后，主进程关闭所有已打开的文件，并向工作进程发送消息要求他们重新打开日志文件。
工作进程收到消息后，也会立刻关闭已打开的日志文件，并重新打开。
这样，旧的文件基本可以立即进行后置处理，例如：压缩。

### Upgrading Executable on the Fly
升级服务器可执行文件，首先需要用新的可执行文件替换旧的可执行文件，然后向主进程发送USR2信号。
主进程收到信号后，首先会将原有的进程文件nginx.pid重命名为nginx.pid.oldbin，然后运行新的可执行文件创建新的工作进程：
```
  PID  PPID USER    %CPU   VSZ WCHAN  COMMAND
33126     1 root     0.0  1164 pause  nginx: master process /usr/local/nginx/sbin/nginx
33134 33126 nobody   0.0  1368 kqread nginx: worker process (nginx)
33135 33126 nobody   0.0  1380 kqread nginx: worker process (nginx)
33136 33126 nobody   0.0  1368 kqread nginx: worker process (nginx)
36264 33126 root     0.0  1148 pause  nginx: master process /usr/local/nginx/sbin/nginx
36265 36264 nobody   0.0  1364 kqread nginx: worker process (nginx)
36266 36264 nobody   0.0  1364 kqread nginx: worker process (nginx)
36267 36264 nobody   0.0  1364 kqread nginx: worker process (nginx)
```
&nbsp;&nbsp;&nbsp;&nbsp;然后所有的工作进程包括老的和旧的都会接收请求。在老的主进程接收到WINCH信号后，老的主进程发送消息要求它所属的所有工作进程优雅退出：
```
  PID  PPID USER    %CPU   VSZ WCHAN  COMMAND
33126     1 root     0.0  1164 pause  nginx: master process /usr/local/nginx/sbin/nginx
33135 33126 nobody   0.0  1380 kqread nginx: worker process is shutting down (nginx)
36264 33126 root     0.0  1148 pause  nginx: master process /usr/local/nginx/sbin/nginx
36265 36264 nobody   0.0  1364 kqread nginx: worker process (nginx)
36266 36264 nobody   0.0  1364 kqread nginx: worker process (nginx)
36267 36264 nobody   0.0  1364 kqread nginx: worker process (nginx)
```
一段时间后只剩下新的工作进程处理请求：
```
  PID  PPID USER    %CPU   VSZ WCHAN  COMMAND
33126     1 root     0.0  1164 pause  nginx: master process /usr/local/nginx/sbin/nginx
36264 33126 root     0.0  1148 pause  nginx: master process /usr/local/nginx/sbin/nginx
36265 36264 nobody   0.0  1364 kqread nginx: worker process (nginx)
36266 36264 nobody   0.0  1364 kqread nginx: worker process (nginx)
36267 36264 nobody   0.0  1364 kqread nginx: worker process (nginx)
```
需要注意的是，老的主进程不会关闭它的监听sockets，如果需要还可以用来创建它自己的工作进程。当新的可执行文件没有按照预期运作时，可以采用以下措施：
* 向老的主进程发送HUP信号。老的主进程会再创建它自己的工作进程，此时不需要再读取配置文件。然后老的主进程会向新的主进程发送QUIT信号，优雅关闭新的主进程下的工作进程。
* 向新的主进程发送TERM信号。新的主进程会发送消息要求所属的所有新的工作进程立刻退出，而对应的工作进程实际也基本如此（如果新的进程由于某些原因，没有退出，那么我们应该使用kill信号指令强制杀死对应进程）。当新的主进程退出后，老的主进程会自动创建自己的工作进程。

如果新的主进程退出服务，老的进程会把已经添加.oldbin后缀的原主进程记录文件名称改回来。
如果升级成功，则会向老的主进程发送QUIT信号，只留下新的主进程：
|PID|PPID|USER|%CPU|VSZ|WCHAN|COMMAND|
|:---:|:---:|:---:|:---:|:---:|:---:|:---|
|36264|1|root|0.0|1148|pause|nginx: master process /usr/local/nginx/sbin/nginx|
|36265|36264|nobody|0.0|1364|kqread|nginx: worker process (nginx)|
|36266|36264|nobody|0.0|1364|kqread|nginx: worker process (nginx)|
|36267|36264|nobody|0.0|1364|kqread|nginx: worker process (nginx)|
36267 36264 nobody   0.0  1364 kqread nginx: worker process (nginx)
