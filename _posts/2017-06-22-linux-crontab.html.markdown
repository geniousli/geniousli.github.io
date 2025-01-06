---
title: Linux crontab
date: 2017-06-22
tags: linux, relearn, crontab
---

linux crontab （例行性工作排程）
--------

##### at
  * 执行一次就结束的程序指令
  * 开启 /etc/init.d/atd restart
  * at任务记录在 /var/spool/at/中
  * 权限 /etc/at.allow, /etc/at.deny, 规则： 1. 先寻找at.allow，写入这个文档的使用者才能使用，没有写入的不能使用 2. 寻找 /etc/at.deny， 写入文档的不能使用， 没有写入的可以使用, 3.两个文档都不存在的，只有root可以使用。
  * 按照上面规则，所以如果是所以使用者都可以使用话，简历空的at.deny就可以了
  * at -l（列出用户的at排程）, -d （取消）, -c （列出id的实际工作内容）
  * 任务管理， atrm id， 移除任务, 等同于 at -d

    ```zsh
      #设定范例
      at HH:MM YYYY-MM-DD
      at HH:MM[am|pm]
      at now + numbers [minutes | hours | days | weeks]
      at now + 5 minutes
      #进入编辑窗口
      ctrl + d 结束
    ```

##### crontab
  * 周期性执行任务
  * crontab -u 只有root才能使用， 帮助其他用户管理任务, -e 编辑， -l 查阅， -r 移除所有工作
  * 分中， 小时， 日期， 月份， 周， 指令（分时日月周）
  * \* 任何时候
  * , 分割枚举
  * - 时间范围
  * /n， 每隔n个单位
  * crontab 每分钟读取， 所以只需要编辑， /etc/crontab 文件即可
  * 如果在大量的crontab时候，出现分配不均，（认为被集中执行）, 则可以，使用分割枚举的方法， 将任务排开，充分利用资源
  * 可以在命令中进行，输出的重定向，例如， /dev/null
  * run-parts, /usr/bin/run-parts, 定期执行script文件
  * /var/log/cron中为crontab的log文件
  * anacron 是可以自动自动检查timestap日志，并将因为开关机导致没有执行的crontab重新执行的程序， 但是需要按照天，月来进行任务排期，

    ```zsh
      # run-parts
      01 * * * * root
      02 4 * * * root
      22 4 * * 0 root
      42 4 1 * * root 分时日月周执行者身仹 挃令串
      root run-parts /etc/cron.hourly <==每小时 run-parts /etc/cron.daily <==每天
      root run-parts /etc/cron.weekly <==每周日 run-parts /etc/cron.monthly <==每个月 1 号
    ```

##### 系统性常见周期任务
  * 登录档的轮替， log rotate
  * 登录文件分析, logwatch
  * locate数据库更新
