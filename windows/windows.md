#### 查看可用指令 ####
```
D:\Program Files\nginx-1.14.0>nginx -h
nginx version: nginx/1.14.0
Usage: nginx [-?hvVtTq] [-s signal] [-c filename] [-p prefix] [-g directives]

Options:
  -?,-h         : this help
  -v            : show version and exit
  -V            : show version and configure options then exit
  -t            : test configuration and exit
  -T            : test configuration, dump it and exit
  -q            : suppress non-error messages during configuration testing
  -s signal     : send signal to a master process: stop, quit, reopen, reload
  -p prefix     : set prefix path (default: NONE)
  -c filename   : set configuration file (default: conf/nginx.conf)
  -g directives : set global directives out of configuration file
```
#### 遇到的问题 ####
```
问题提示：1113: No mapping for the Unicode character exists in the target multi-byte code page
原因：解压的路径里面包含有中文
```
#### 常用指令 ####
```
start nginx      --后台运行，一闪而过
nginx -c conf/custom/my.conf      --显式y
nginx -s stop       --关闭nginx服务
nginx -s quit       --优雅关闭
```
#### 个人配置 ####
```
请查看conf/custom/文件夹下。
主文件my.conf，include以下文件：
  mime.types拷贝自conf方便使用；
  default.conf保存原有默认配置；
  proxyBaidu.conf配置代理百度；
  proxyLocal配置代理本地。
```
