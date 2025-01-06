---
title: Linux Shell
date: 2017-06-19
tags: linux, relearn, bash, shell
---

linux shell
--------

#### 常用命令
  * history, alias, type, 变量，echo, unset, env, declare, set, read, array
  * 系统变量， HOME, SHELL, HISISIZE, PATH, LANG, RANDOM, $, $?, OSTYPE, HOSTTYPE, MACHTYPE,
  * read, read -p 'please enter you name' -t 100 name
  * declare, declare -a(array) ary; declear (-i) integer, declear -x(设定为幻境变量), declare -r (read only); ary[1]='xx' 从1开始，而非从0开始
  * ulimit 用于限制用户的系统资源，可以打开的档案数量， cpu时间， 内存总量等
  * alias, unalias,  alias rm='rm i'
  * history 10
  * 指定的搜索顺序， type -a ls, 1. 以相对/绝对路径执行挃令,例如『/bin/ls』戒『./ls』; 2. 由alias找刡该挃令杢执行; 3. 由bash内建癿(builtin)挃令杢执行; 4. 透过$PATH这个发量癿顺序搜寻刡癿第一个挃令杢执行。   
  * bash 的登录欢迎信息， /etc/issue, /etc/motd, /etc/issue.net, 远程sshd登录，可以配置/etc/ssh/sshd_config 来决定， 读取文件   
  * bash 的配置文件， login 读取文件顺序， 1. /etc/profile, 2. ~/.bash_profie, ~/.bash_login, ~/.profile , 其中2中三选一， 按照顺序那个文件存在读取那个文件 1。中不要改动比较重要， 1中会另外呼入 /etc/profile.d/\*.sh 文件夹夏的所有文件， /etc/profile.d/lang.sh 会呼入/etc/sysconfig/i18n文件, /etc/skel/.bashrc 文件是 ~/.bashrc 的备份文件
  * source=. 可以source ~/.bash_profile 或者 . ~/.bash_profile
  * ~/.bash_history, ~/.bash_logout
  * 输入输出流管理：

    >
      1. 标准输入 (stdin):代码为0,使用<戒<<;
      2. 标准输出 (stdout):代码为1,使用>戒>>;
      3. 标准错诨输出(stderr):代码为2,使用2>戒2>>;
     find /home -name .bashrc > list_right 2> list_error
     /dev/null 垃圾桶黑洞装置不特殊写法
     find /home -name .bashrc > list 2>&1 同时写入一个文件中，不能写成 > list 2> list会导致同时吸入问津啊，顺序发生错乱
     command1 && command2 || command3

  * cut; cut -d ':' -f 2 $PATH; 通过分隔符裁剪，取特定位置的数值
  * expand -t 4 file_name, 将tab转换成4个空格
  * split -b 10m file 将文件切分成10m大小
  * xargs, sort, uniq

总结
------------

  * 由于核心在内存中是受保护,因此我们必须要透过 Shell将我们输入命令传给 Kernel 沟通
  * 用户默认登入去的shell记录于的最后一个字段
  * bash 主要功能有： 工作控制，前景背景控制，程序脚本化(job control, foreground, background)
  * type, which
  * locale 查询目前语言系材料， /etc/sysconfig/i18n
  * read可以回去用户输入，交互
  * ulimit限制用户系统资源
  * bash 非为login shell， non-login shell， login shell主要读取 /etc/profile, ~/.bash_profile, non-login shell 仅读取 ~/.bashrc
  * 命令集， cut, grep, sort, wc, uniq, tee, tr, col, join, paste, expand, split, xargs
  * shell中的提示字符， 修改PSI变量
  * 登录欢迎信息，在 /etc/issue, /etc/motd



  * 变量：

    >
      name=file_name
      name='file name is --'
      name="$name":suffix; name=${name}:suffix;
      name=$(uname -r); name=`unname -r`
