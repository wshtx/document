# shell

### 基础

- 学会如何使用 man 命令去阅读文档。学会使用 apropos 去查找文档。知道有些命令并不对应可执行文件，而是在 Bash 内置好的，此时可以使用 help 和 help -d 命令获取帮助信息。你可以用 type 命令 来判断这个命令到底是可执行文件、shell 内置命令还是别名。
- 学会使用 > 和 < 来重定向输出和输入，学会使用 | 来重定向管道。明白 > 会覆盖了输出文件而 >> 是在文件末添加。了解标准输出 stdout 和标准错误 stderr。
- 学会使用通配符 * （或许再算上 ? 和 [...]） 和引用以及引用中 ' 和 " 的区别（后文中有一些具体的例子）。
- 熟悉 Bash 中的任务管理工具：&，ctrl-z，ctrl-c，jobs，fg，bg，kill 等。
- 学会使用 ssh 进行远程命令行登录，最好知道如何使用 ssh-agent，ssh-add 等命令来实现基础的无密码认证登录。
- 学会基本的文件管理工具：ls 和 ls -l （了解 ls -l 中每一列代表的意义），less，head，tail 和 tail -f （甚至 less +F），ln 和 ln -s （了解硬链接与软链接的区别），chown，chmod，du （硬盘使用情况概述：du -hs *）。 关于文件系统的管理，学习 df，mount，fdisk，mkfs，lsblk。知道 inode 是什么（与 ls -i 和 df -i 等命令相关）。
- 学习基本的网络管理工具：ip 或 ifconfig，dig。
- 学习并使用一种版本控制管理系统，例如 git。
- 熟悉正则表达式，学会使用 grep／egrep，它们的参数中 -i，-o，-v，-A，-B 和 -C 这些是很常用并值得认真学习的。
- 学会使用 apt-get，yum，dnf 或 pacman （具体使用哪个取决于你使用的 Linux 发行版）来查找和安装软件包。并确保你的环境中有 pip 来安装基于 Python 的命令行工具 （接下来提到的部分程序使用 pip 来安装会很方便）。

### 日常使用

- 在 Bash 中，可以通过按 Tab 键实现自动补全参数，使用 ctrl-r 搜索命令行历史记录（按下按键之后，输入关键字便可以搜索，重复按下 ctrl-r 会向后查找匹配项，按下 Enter 键会执行当前匹配的命令，而按下右方向键会将匹配项放入当前行中，不会直接执行，以便做出修改）。
- 在 Bash 中，可以按下 ctrl-w 删除你键入的最后一个单词，ctrl-u 可以删除行内光标所在位置之前的内容，alt-b 和 alt-f 可以以单词为单位移动光标，ctrl-a 可以将光标移至行首，ctrl-e 可以将光标移至行尾，ctrl-k 可以删除光标至行尾的所有内容，ctrl-l 可以清屏。键入 man readline 可以查看 Bash 中的默认快捷键。内容有很多，例如 alt-. 循环地移向前一个参数，而 alt-* 可以展开通配符。
- 为了便于编辑长命令，在设置你的默认编辑器后（例如 export EDITOR=vim），ctrl-x ctrl-e 会打开一个编辑器来编辑当前输入的命令。在 vi 风格下快捷键则是 escape-v。
- 键入 history 查看命令行历史记录，再用 !n（n 是命令编号）就可以再次执行。其中有许多缩写，最有用的大概就是 !$， 它用于指代上次键入的参数，而 !! 可以指代上次键入的命令了（参考 man 页面中的“HISTORY EXPANSION”）。不过这些功能，你也可以通过快捷键 ctrl-r 和 alt-. 来实现。、cd 命令可以切换工作路径，输入 cd ~ 可以进入 home 目录。要访问你的 home 目录中的文件，可以使用前缀 ~（例如 ~/.bashrc）。在 sh 脚本里则用环境变量 $HOME 指代 home 目录的路径。
- 回到前一个工作路径：cd -。
- 如果你输入命令的时候中途改了主意，按下 alt-# 在行首添加 # 把它当做注释再按下回车执行（或者依次按下 ctrl-a， #， enter）。这样做的话，之后借助命令行历史记录，你可以很方便恢复你刚才输入到一半的命令。
- `pstree -p` 以一种优雅的方式展示进程树。
- 使用 `pgrep` 和 `pkill` 根据名字查找进程或发送信号（`-f` 参数通常有用，匹配完整指令名）
- 了解你可以发往进程的信号的种类。比如，使用 `kill -STOP [pid]` 停止一个进程。使用 `man 7 signal` 查看详细列表。
- 使用 `nohup` 或 `disown` 使一个后台进程持续运行。
- 使用 `netstat -lntp` 或 `ss -plat` 检查哪些进程在监听端口（默认是检查 TCP 端口; 添加参数 `-u` 则检查 UDP 端口）或者 `lsof -iTCP -sTCP:LISTEN -P -n` (这也可以在 OS X 上运行)。
- `lsof` 来查看开启的套接字和文件。
- 使用 `uptime` 或 `w` 来查看系统已经运行多长时间。
- 使用 `alias` 来创建常用命令的快捷形式。例如：`alias ll='ls -latr'` 创建了一个新的命令别名 `ll`。
- 可以把别名、shell 选项和常用函数保存在 `~/.bashrc`，具体看下这篇[文章](http://superuser.com/a/183980/7106)。这样做的话你就可以在所有 shell 会话中使用你的设定。
- 把环境变量的设定以及登陆时要执行的命令保存在 `~/.bash_profile`。而对于从图形界面启动的 shell 和 `cron` 启动的 shell，则需要单独配置文件。
- 要想在几台电脑中同步你的配置文件（例如 `.bashrc` 和 `.bash_profile`），可以借助 Git。
- 当变量和文件名中包含空格的时候要格外小心。Bash 变量要用引号括起来，比如 `"$FOO"`。尽量使用 `-0` 或 `-print0` 选项以便用 NULL 来分隔文件名，例如 `locate -0 pattern | xargs -0 ls -al` 或 `find / -print0 -type d | xargs -0 ls -al`。如果 for 循环中循环访问的文件名含有空字符（空格、tab 等字符），只需用 `IFS=$'\n'` 把内部字段分隔符设为换行符。
- 在 Bash 脚本中，使用 `set -x` 去调试输出（或者使用它的变体 `set -v`，它会记录原始输入，包括多余的参数和注释）。尽可能地使用严格模式：使用 `set -e` 令脚本在发生错误时退出而不是继续运行；使用 `set -u` 来检查是否使用了未赋值的变量；试试 `set -o pipefail`，它可以监测管道中的错误。当牵扯到很多脚本时，使用 `trap` 来检测 ERR 和 EXIT。一个好的习惯是在脚本文件开头这样写，这会使它能够检测一些错误，并在错误发生时中断程序并输出信息：

```
      set -euo pipefail
      trap "echo 'error: Script failed: see failed command above'" ERR
```

- 在 Bash 脚本中，子 shell（使用括号 `(...)`）是一种组织参数的便捷方式。一个常见的例子是临时地移动工作路径，代码如下：

```
      # do something in current dir
      (cd /some/other/dir && other-command)
      # continue in original dir
```

- 在 Bash 中，变量有许多的扩展方式。`${name:?error message}` 用于检查变量是否存在。此外，当 Bash 脚本只需要一个参数时，可以使用这样的代码 `input_file=${1:?usage: $0 input_file}`。在变量为空时使用默认值：`${name:-default}`。如果你要在之前的例子中再加一个（可选的）参数，可以使用类似这样的代码 `output_file=${2:-logfile}`，如果省略了 $2，它的值就为空，于是 `output_file` 就会被设为 `logfile`。数学表达式：`i=$(( (i + 1) % 5 ))`。序列：`{1..10}`。截断字符串：`${var%suffix}` 和 `${var#prefix}`。例如，假设 `var=foo.pdf`，那么 `echo ${var%.pdf}.txt` 将输出 `foo.txt`。
- 使用括号扩展（`{`...`}`）来减少输入相似文本，并自动化文本组合。这在某些情况下会很有用，例如 `mv foo.{txt,pdf} some-dir`（同时移动两个文件），`cp somefile{,.bak}`（会被扩展成 `cp somefile somefile.bak`）或者 `mkdir -p test-{a,b,c}/subtest-{1,2,3}`（会被扩展成所有可能的组合，并创建一个目录树）。
- 通过使用 `<(some command)` 可以将输出视为文件。例如，对比本地文件 `/etc/hosts` 和一个远程文件：

```
      diff /etc/hosts <(ssh somehost cat /etc/hosts)
```

- 编写脚本时，你可能会想要把代码都放在大括号里。缺少右括号的话，代码就会因为语法错误而无法执行。如果你的脚本是要放在网上分享供他人使用的，这样的写法就体现出它的好处了，因为这样可以防止下载不完全代码被执行。

```
{
      # 在这里写代码
}
```

- 了解 Bash 中的“here documents”，例如 `cat <<EOF ...`。
- 在 Bash 中，同时重定向标准输出和标准错误：`some-command >logfile 2>&1` 或者 `some-command &>logfile`。通常，为了保证命令不会在标准输入里残留一个未关闭的文件句柄捆绑在你当前所在的终端上，在命令后添加 `</dev/null` 是一个好习惯。
- 使用 `man ascii` 查看具有十六进制和十进制值的ASCII表。`man unicode`，`man utf-8`，以及 `man latin1` 有助于你去了解通用的编码信息。
- 使用 `screen` 或 [`tmux`](https://tmux.github.io/) 来使用多份屏幕，当你在使用 ssh 时（保存 session 信息）将尤为有用。而 `byobu` 可以为它们提供更多的信息和易用的管理工具。另一个轻量级的 session 持久化解决方案是 [`dtach`](https://github.com/bogner/dtach)。
- ssh 中，了解如何使用 `-L` 或 `-D`（偶尔需要用 `-R`）开启隧道是非常有用的，比如当你需要从一台远程服务器上访问 web 页面。
- 对 ssh 设置做一些小优化可能是很有用的，例如这个 `~/.ssh/config` 文件包含了防止特定网络环境下连接断开、压缩数据、多通道等选项：

```
      TCPKeepAlive=yes
      ServerAliveInterval=15
      ServerAliveCountMax=6
      Compression=yes
      ControlMaster auto
      ControlPath /tmp/%r@%h:%p
      ControlPersist yes
```

- 一些其他的关于 ssh 的选项是与安全相关的，应当小心翼翼的使用。例如你应当只能在可信任的网络中启用 `StrictHostKeyChecking=no`，`ForwardAgent=yes`。
- 考虑使用 [`mosh`](https://mosh.mit.edu/) 作为 ssh 的替代品，它使用 UDP 协议。它可以避免连接被中断并且对带宽需求更小，但它需要在服务端做相应的配置。
- 获取八进制形式的文件访问权限（修改系统设置时通常需要，但 `ls` 的功能不那么好用并且通常会搞砸），可以使用类似如下的代码：

```
      stat -c '%A %a %n' /etc/timezone
```

- 使用 [`percol`](https://github.com/mooz/percol) 或者 [`fzf`](https://github.com/junegunn/fzf) 可以交互式地从另一个命令输出中选取值。
- 使用 `fpp`（[PathPicker](https://github.com/facebook/PathPicker)）可以与基于另一个命令(例如 `git`）输出的文件交互。
- 将 web 服务器上当前目录下所有的文件（以及子目录）暴露给你所处网络的所有用户，使用： `python -m SimpleHTTPServer 7777` （使用端口 7777 和 Python 2）或`python -m http.server 7777` （使用端口 7777 和 Python 3）。
- 以其他用户的身份执行命令，使用 `sudo`。默认以 root 用户的身份执行；使用 `-u` 来指定其他用户。使用 `-i` 来以该用户登录（需要输入_你自己的_密码）。
- 将 shell 切换为其他用户，使用 `su username` 或者 `su - username`。加入 `-` 会使得切换后的环境与使用该用户登录后的环境相同。省略用户名则默认为 root。切换到哪个用户，就需要输入_哪个用户的_密码。
- 了解命令行的 [128K 限制](https://wiki.debian.org/CommonErrorMessages/ArgumentListTooLong)。使用通配符匹配大量文件名时，常会遇到“Argument list too long”的错误信息。（这种情况下换用 `find` 或 `xargs` 通常可以解决。）
- 当你需要一个基本的计算器时，可以使用 `python` 解释器（当然你要用 python 的时候也是这样）。例如：

```
>>> 2+3
5
```

- 
