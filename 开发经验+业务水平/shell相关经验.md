# 把ssh/sftp目录挂载为windows目录或驱动器

详见[Windows SSHFS/SFTP mounting clients](https://softwarerecs.stackexchange.com/questions/13875/windows-sshfs-sftp-mounting-clients).

可用的解决方案

1. 使用开源免费的doran+sshfs.
2. 使用约$75获得ExpanDrive的终生个人账号.
3. 使用非商用免费的[Sftp Netdrive](http://www.sftpnetdrive.com/).


# 行尾^M导致脚本执行失败

通常windows下文本文件的行尾为\r\n, 其中\r就是^M. 当windows的文本文件直接复制到unix中, 并且当做sh脚本、makefile、python脚本等执行时, 会报语法错误, 而且报错信息中含`^M`字样.

此时我们可以用vim打开脚本文件, 执行

```sh
:set fileformat=unix
:w
```

文件的行尾就转换成unix格式了.

# bash的比较和条件分支

详见[bash编程之if-else条件判断](http://zhaochj.blog.51cto.com/368705/1315581).

这个博客的排版很工整, 最重要的是, 空格和缩进完全正确. 例子也很生动.