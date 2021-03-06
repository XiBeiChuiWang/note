# Linux（一）

1. ### 入门概述

   - #### Linux 简介

     Linux 内核最初只是由芬兰人林纳斯·托瓦兹（Linus Torvalds）在赫尔辛基大学上学时出于个人爱好而编写的。

     Linux 是一套免费使用和自由传播的类 Unix 操作系统，是一个基于 POSIX（可移植操作系统接口） 和UNIX 的多用户、多任务、支持多线程和多 CPU 的操作系统。

     Linux 能运行主要的 UNIX 工具软件、应用程序和网络协议。它支持 32 位和 64 位硬件。Linux 继承了Unix 以网络为核心的设计思想，是一个性能稳定的多用户网络操作系统。

   - #### Linux 发行版

      Linux 的发行版说简单点就是将 Linux 内核与应用软件做一个打包。 

   - #### Linux 应用领域

     今天各种场合都有使用各种 Linux 发行版，从嵌入式设备到超级计算机，并且在服务器领域确定了地位，通常服务器使用 LAMP（Linux + Apache + MySQL + PHP）或 LNMP（Linux + Nginx+ MySQL +PHP）组合。

     目前 Linux 不仅在家庭与企业中使用，并且在**中也很受欢迎。

     - 巴西联邦由于支持 Linux 而世界闻名。
     - 有新闻报道俄罗斯军队自己制造的 Linux 发布版的，做为 G.H.ost 项目已经取得成果。
     - 印度的 Kerala 联邦计划在向全联邦的高中推广使用 Linux。
     - 中华人民共和国为取得技术独立，在龙芯处理器中排他性地使用 Linux。
     - 在西班牙的一些地区开发了自己的 Linux 发布版，并且在**与教育领域广泛使用，如Extremadura 地区的 gnuLinEx 和 Andalusia 地区的 Guadalinex。
     - 葡萄牙同样使用自己的 Linux 发布版 Caixa Mágica，用于 Magalh?es 笔记本电脑和 e-escola 软件。
     - 法国和德国同样开始逐步采用 Linux。

2. ### 走近Linux系统

   - #### 开机登录

     开机会启动许多程序。它们在Windows叫做”服务”（service），在Linux就叫做”守护进程”（daemon）。

     开机成功后，它会显示一个文本登录界面，这个界面就是我们经常看到的登录界面，在这个登录界面中会提示用户输入用户名，而用户输入的用户将作为参数传给login程序来验证用户的身份，密码是不显示的，输完回车即可！

     一般来说，用户的登录方式有三种：

     - 命令行登录
     - ssh登录
     - 图形界面登录

     最高权限账户为 root，可以操作一切！

   - #### 关机

     在linux领域内大多用在服务器上，很少遇到关机的操作。毕竟服务器上跑一个服务是永无止境的，除非特殊情况下，不得已才会关机。

     关机指令为：shutdown ；

     ```bash
     sync # 将数据由内存同步到硬盘中。
     shutdown # 关机指令，你可以man shutdown 来看一下帮助文档。例如你可以运行如下命令关机：
     shutdown –h 10 # 这个命令告诉大家，计算机将在10分钟后关机
     shutdown –h now # 立马关机
     shutdown –h 20:25 # 系统会在今天20:25关机
     shutdown –h +10 # 十分钟后关机
     shutdown –r now # 系统立马重启
     shutdown –r +10 # 系统十分钟后重启
     reboot # 就是重启，等同于 shutdown –r now
     halt # 关闭系统，等同于shutdown –h now 和 poweroff
     ```

   - #### 系统目录结构

     -  一切皆文件 
     -  根目录 / ，所有的文件都挂载在这个节点下 
     - 主要目录
       - /bin： bin是Binary的缩写, 这个目录存 放着最经常使用的命令。
       - /boot： 这里存放的是启动Linux时使用的一些核心文件，包括一些连接文件以及镜像文件。（不要动）
       - /dev ： dev是Device(设备)的缩写, 存放的是Linux的外部设备，在Linux中访问设备的方式和访问文件的方式是相同的。
       - ==/etc： 这个目录用来存放所有的系统管理所需要的配置文件和子目录。==
       - ==/home：用户的主目录，在Linux中，每个用户都有一个自己的目录，一般该目录名是以用户的账号命名的。==
       - /lib： 这个目录里存放着系统最基本的动态连接共享库，其作用类似于Windows里的DLL文件。（不要动）
       - /lost+found： 这个目录一般情况下是空的，当系统非法关机后，这里就存放了一些文件。（存放突然关机的一些文件）
       - /media：linux系统会自动识别一些设备，例如U盘、光驱等等，当识别后，linux会把识别的设备挂载到这个目录下。
       - /mnt：系统提供该目录是为了让用户临时挂载别的文件系统的，我们可以将光驱挂载在/mnt/上，然后进入该目录就可以查看光驱里的内容了。（我们后面会把一些本地文件挂载在这个目录下）
       - ==/opt：这是给主机额外安装软件所摆放的目录。比如你安装一个ORACLE数据库则就可以放到这个目录下。默认是空的。==
       - /proc： 这个目录是一个虚拟的目录，它是系统内存的映射，我们可以通过直接访问这个目录来获取系统信息。（不用管）
       - ==/root：该目录为系统管理员，也称作超级权限者的用户主目录。==
       - /sbin：s就是Super User的意思，这里存放的是系统管理员使用的系统管理程序。
       - /srv：该目录存放一些服务启动之后需要提取的数据。
       - /sys：这是linux2.6内核的一个很大的变化。该目录下安装了2.6内核中新出现的一个文件系统sysfs 。
       - ==/tmp：这个目录是用来存放一些临时文件的。用完即丢的文件，可以放在这个目录下，安装包！==
       - ==/usr：这是一个非常重要的目录，用户的很多应用程序和文件都放在这个目录下，类似于windows下的program files目录。==
       - /usr/bin： 系统用户使用的应用程序。
       - /usr/sbin： 超级用户使用的比较高级的管理程序和系统守护程序。Super
       - /usr/src： 内核源代码默认的放置目录。
       - /var：这个目录中存放着在不断扩充着的东西，我们习惯将那些经常被修改的目录放在这个目录下。包括各种日志文件。
       - /run：是一个临时文件系统，存储系统启动以来的信息。当系统重启时，这个目录下的文件应该被删掉或清除。
       - /www：存放服务器网站相关的资源，环境，网站的项目

3. ### 常用基本命令

   1. #### 目录管理

      - ##### 绝对路径、相对路径

        - cd ： 切换目录命令！
        - ./ ： 当前目录
        - cd .. ： 返回上一级目录

      - ##### ls（列出目录）

        在Linux中 ls 可能是最常常被使用的 !

        - -a参数：all ，查看全部的文件，包括隐藏文件
        - -l 参数 列出所有的文件，包含文件的属性和权限，没有隐藏文件

        所有Linux命令可以组合使用！

      - ##### cd 命令 切换目录

         cd 目录名（绝对路径都是以 / 开头，相对路径，对于当前目录该如何寻找 ../../） 

      - ##### pwd 显示当前用户所在的目录！

      - 创建目录

        - mkdir xxx ： 创建目录
        - mkdir -p test2/test3/test4： 创建多级目录

      - 删除目录

        - rmdir xxx ： 删除不为空的目录
        - rmdir -p test2/test3/test4 ：强制删除目录

         rmdir 仅能删除空的目录，如果下面存在文件，需要先删除文件，递归删除多个目录 -p 参数即可 

      - ##### cp (复制文件或者目录)

        cp 原来的地方 新的地方

        - 如果文件重复，就选则覆盖（y）或者 放弃（n）

      - ##### rm （移除文件或者目录！）

        ```bash
        rm -rf / # 系统中所有的文件就被删除了，删库跑路就是这么操作的！
        ```

      - ##### mv 移动文件或者目录！重命名文件

        - -f 强制
        - -u 只替换已经更新过的文件
        - mv install.sh kuangstudy/ 移动文件
        - mv a b  重命名文件夹

   2. #### 基本属性

      Linux系统是一种典型的多用户系统，不同的用户处于不同的地位，拥有不同的权限。为了保护系统的安全性，Linux系统对不同的用户访问同一文件（包括目录文件）的权限做了不同的规定。

      在Linux中我们可以使用 `ll` 或者 `ls –l` 命令来显示一个文件的属性以及文件所属的用户和组，如

      ![](https://pic.imgdb.cn/item/6038ef895f4313ce2548268b.png)

      在Linux中第一个字符代表这个文件是目录、文件或链接文件等等：

      - ==当为[ d ]则是目录==
      - ==当为[ - ]则是文件；==
      - ==若是[ l ]则表示为链接文档 ( link file )；==
      - 若是[ b ]则表示为装置文件里面的可供储存的接口设备 ( 可随机存取装置 )；
      - 若是[ c ]则表示为装置文件里面的串行端口设备，例如键盘、鼠标 ( 一次性读取装置 )。

      接下来的字符中，以三个为一组，且均为『rwx』 的三个参数的组合。

      其中，[ r ]代表可读(read)、[ w ]代表可写(write)、[ x ]代表可执行(execute)。

      要注意的是，这三个权限的位置不会改变，如果没有权限，就会出现减号[ - ]而已。

      每个文件的属性由左边第一部分的10个字符来确定：

      从左至右用0-9这些数字来表示。

      第0位确定文件类型，第1-3位确定属主（该文件的所有者）拥有该文件的权限。第4-6位确定属组（所有者的同组用户）拥有该文件的权限，第7-9位确定其他用户拥有该文件的权限。

      其中：

      - 第1、4、7位表示读权限，如果用”r”字符表示，则有读权限，如果用”-“字符表示，则没有读权限；
      - 第2、5、8位表示写权限，如果用”w”字符表示，则有写权限，如果用”-“字符表示没有写权限；
      - 第3、6、9位表示可执行权限，如果用”x”字符表示，则有执行权限，如果用”-“字符表示，则没有执行权
        限。

      对于文件来说，它都有一个特定的所有者，也就是对该文件具有所有权的用户。

      同时，在Linux系统中，用户是按组分类的，一个用户属于一个或多个组。

      文件所有者以外的用户又可以分为文件所有者的同组用户和其他用户。

      因此，Linux系统按文件所有者、文件所有者同组用户和其他用户来规定了不同的文件访问权限。

      在以上实例中，boot 文件是一个目录文件，属主和属组都为 root。

      - ##### 修改文件属性

        1、chgrp：更改文件属组

        ```
        chgrp [-R] 属组名 文件名
        ```

        -R：递归更改文件属组，就是在更改某个目录文件的属组时，如果加上-R的参数，那么该目录下的所有文件的属组都会更改。

        2、chown：更改文件属主，也可以同时更改文件属组

        ```
        chown [–R] 属主名 文件名chown [-R] 属主名：属组名 文件名
        ```

        3、chmod：更改文件9个属性（必须要掌握）

        你没有权限操作此文件！

        ```
        chmod [-R] xyz 文件或目录
        ```

        Linux文件属性有两种设置方法，一种是数字（常用的是数字），一种是符号。

        Linux文件的基本权限就有九个，分别是owner/group/others三种身份各有自己的read/write/execute权限。

        先复习一下刚刚上面提到的数据：文件的权限字符为：『-rwxrwxrwx』， 这九个权限是三个三个一组的！其中，我们可以使用数字来代表各个权限，各权限的分数对照表如下：

        ```
        r:4    w:2    x:1可读可写不可执行 rw- 6可读可写不课执行 rwx 7chomd  777  文件赋予所有用户可读可执行！
        ```

        每种身份(owner/group/others)各自的三个权限(r/w/x)分数是需要累加的，例如当权限为：[-rwxrwx—-]分数则是：

        - owner = rwx = 4+2+1 = 7
        - group = rwx = 4+2+1 = 7
        - others= --- = 0+0+0 = 0

      - ##### 文件内容查看

        我们会经常使用到文件查看！

        Linux系统中使用以下命令来查看文件的内容：

        - cat 由第一行开始显示文件内容，用来读文章，或者读取配置文件啊，都使用cat名
        - tac 从最后一行开始显示，可以看出 tac 是 cat 的倒着写！
        -  nl 显示的时候，顺道输出行号！ 看代码的时候，希望显示行号！ 常用 
        -  more 一页一页的显示文件内容，带余下内容的（空格代表翻页，enter 代表向下看一行， :f 行号） 
        -  less 与 more 类似，但是比 more 更好的是，他可以往前翻页！ （空格下翻页，pageDown，pageUp键代表翻动页面！退出 q 命令，查找字符串 /要查询的字符向下查询，向上查询使用？要查询的字符串，n 继续搜寻下一个，N 上寻找！） 
        -  head 只看头几行 通过 -n 参数来控制显示几行！ 
        -  tail 只看尾巴几行 -n 参数 要查看几行！ 