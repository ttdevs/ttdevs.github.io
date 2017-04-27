
## 0x00 场景

>  Nginx ＋ uWSGI ＋ Flask 的场景中，当我们的代码出现异常或者其他一些原因导致服务挂掉，我们会希望能自动重启 `uWSGI`

这个时候我们我们可以考虑使用 `supervisor`来达到我们的目的。此文接上一篇，因此我的系统还是Raspbian(Debian)。这里分享一些简单的配置使用。先看一下 `supervisor` 的介绍：

> [Supervisor is a client/server system that allows its users to monitor and control a number of processes on UNIX-like operating systems.](http://supervisord.org/)


## 0x01 安装

这个比较简单，我们直接使用 `apt-get` 即可安装。

`pi@raspberrypi:~ $ sudo apt-get install supervisor` 

(卸载用：apt-get remove)

当然，你也可以用 `pip` 进行安装：

`sudo pip install supervisor`


## 0x02 配置

`supervisor` 的配置文件位于 `/etc/supervisor/` 下，如：

``` shell
pi@raspberrypi:/etc/supervisor $ ls
conf.d  supervisord.conf
```

`supervisord.conf` 是默认的配置文件，我们无需修改，在 `conf.d` 目录下，我们创建自己的配置文件即可，这里我建了一个 `uwsgi.conf`  文件：

``` shell
pi@raspberrypi:/etc/supervisor $ cd conf.d/
pi@raspberrypi:/etc/supervisor/conf.d $ touch uwsgi.conf
pi@raspberrypi:/etc/supervisor/conf.d $ sudo nano uwsgi.conf
```

`uwsgi.conf` 文件内容：

``` shell
[program:raspi]
command=/home/pi/raspi/venv/bin/uwsgi /home/pi/raspi/uwsgi.ini
directory=/home/pi/raspi
user=pi
autostart=true
autorestart=true
stdout_logfile=/home/pi/raspi/logs/uwsgi_supervisor.log
stderr_logfile=/home/pi/raspi/logs/uwsgi_supervisor_err.log
```
- command：执行的命令
- directory：工作的目录，在执行前会进行切换，由于上面我使用的绝对路径，可以不配置
- stdout_logfile 和 stderr_logfile 配置我们日志输出位置

 这个配置文件的更多可选参数可参考[这里](http://supervisord.org/configuration.html#program-x-section-settings)。
 
`supervisor` 还提供了一个web管理界面，我们需要先对其进行配置，修改 `/etc/supervisor/supervisord.conf` 文件，增加如下内容即可：
 
 ``` shell
 [supervisorctl]
serverurl=unix:///var/run/supervisor.sock ; use a unix:// URL  for a unix socket
serverurl=http://192.168.1.56:9000
username=ttdevs            ; (default is no username (open server))
password=admin             ; (default is no password (open server))
; The [include] section can just contain the "files" setting.  This
; setting can list multiple files (separated by whitespace or
; newlines).  It can also contain wildcards.  The filenames are
; interpreted as relative to this file.  Included files *cannot*
; include files themselves.

[include]
files = /etc/supervisor/conf.d/*.conf

[inet_http_server]         ; inet (TCP) server disabled by default
port=192.168.1.56:9000 	           ; (ip_address:port specifier, *:port for all iface)
username=ttdevs            ; (default is no username (open server))
password=admin             ; (default is no password (open server))
 ```


## 0x03 操作

安装好以后，我们有两个命令可以使用，一个是 `supervisord`，另一个是 `supervisorctl`，`supervisord` 负责启动服务，`supervisorctl` 负责操作服务，C/S模式。

- 启动 `supervisor`： `sudo supervisord`

	更多的启动参数可以参考[这里](http://supervisord.org/running.html#supervisord-command-line-options)。
	
- 启动/重启/停止某个我们配置的进程：`supervisorctl start/restart/stop programABC`

另外在上面的配置中，我们还可以使用浏览器访问 `http://192.168.1.56:9000/`对我们的进程进行管理。


## 0xFF 参考

1. http://blog.chinaunix.net/uid-26000296-id-4759916.html
2. http://liyangliang.me/posts/2015/06/using-supervisor/
3. http://www.restran.net/2015/10/04/supervisord-tutorial/

![Create by ttdevs](https://raw.githubusercontent.com/ttdevs/ttdevs.github.io/common/images/logo.png)


