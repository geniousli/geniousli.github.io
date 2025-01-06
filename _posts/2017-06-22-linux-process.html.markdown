---
title: Linux process
date: 2017-06-22
tags: linux, relearn, process
---

linux process
--------

##### 程序
  * 触发任何一个事件，系统会将他定义为一个程序，赋予PID，根据调用用户与相关属性关系，赋予PID相关的有效的权限设定， 在系统上进行的动作就与这个权限相关
  * shell是一个程序， 执行bash， 与内核交互
  * fork and exec, fork, 父进程 到子进程， 赋予子进程新的PID， PPID=父进程， exec 程序实体
  * crontab, atd, syslog, 为常驻进程
#### 工作管理
  * background job, foreground job
  * background job 中 的限制
    1. 为当前bash的子进程
    2. 可以自行运作，无法使用ctrl+c终止， 只能使用fg/bg
    3. 不能够与终端交互

  * 丢到背景执行 & tar -zpcf /tmp/etc.tar.gz /etc &
  * ctrl+z 讲当前工作丢到背景执行 并暂停, 例如 vi ~/.bashrc
  * jobs 查询背景执行工作状态 -l 列表， -r running的工作， -s stop 的工作
  * fg jobnunber， 将背景执行转移到前景执行
  * bg jobnunber， 将前景执行转移背景执行
  * kill -signal %jobnumber
  * ps aux(系统所有程序), -lA(同aux) axjf（连同程序树状态）
  * ps -l 仅查找bash相关程序, 列表中个个字段的含义 F, (process flags),4 为root， 1为此进程fork的， S(state)， R running， S sleep, D 不可唤醒状态, T 停止状态， Z 僵尸状态, UID/PID,PPID(parent pid), PRI/NI(priority/nice) 程序的优先级
  * top, 动态查看程序的变化，-p pid -d 几秒刷新
  * pstree -p pid, -u process 所属帐号
  * kill 中signal, kill -signal PID
    * 1, SIGHUP, 重新读取配置文件，重新启动
    * 2, SIGINT, 终止程序
    * 9, SIGKILL 强制终止
    * 15, SIGTERM 正常结束程序
    * 17, SIGSTOP，暂停程序的执行
  * 程序中的优先级，sum = PRI + nice, pri为动态计算的， nice为人工指定，范围为(-20 - 19 )
       > nice -n command, renice number PID

  * free 查看内存使用情况,  free -b m(Mbytes) b(bytes), -t 显示swap总量(很有效率的使用没存， 比如使用大量buffer， cached)
  * uname 查看系统与核心相关程序
  * uptime, 观察系统启动时间与工作负载
  * netstat 追踪网络, -a（所有联机， 监听， socket列出来）, -t (tcp网络封包数据), -u（udp网络封包数据） -n port number， -l 列出正在监听的服务， -p pid
  * fuser:藉由档案(戒文件系统)找出正在使用该档案癿程序, 找出正在占用文件资源的程序，-u 列出使用者，-v 列出程序指令， -k， 住处pid，并试图发送SIGKILL信号， -i交互式发送信号
  * pidof -x 列出该program name的可能的PPID的pid（找出父进程关联的pid） program_name
