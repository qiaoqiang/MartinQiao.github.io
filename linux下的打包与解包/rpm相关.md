# RPM包
## 概述
RPM是Red-Hat Package Manager（RPM软件包管理器）的缩写，这一文件格式名称虽然打上了RedHat的标志，
但是其原始设计理念是开放式的，现在包括OpenLinux、S.u.S.E.以及Turbo Linux等Linux的分发版本都有采用，可以算是公认的行业标准了。


## rpm管理工具命令
1.rpm
安装： rpm -ivh 1.rpm


作为一个软件包管理工具，RPM管理着系统已安装的所有RPM程序组件的资料。我们也可以使用RPM来卸载相关的应用程序。
rpm －e xv
RPM的常用参数还包括：
－vh：显示安装进度；
－U：升级软件包；
－qpl：列出RPM软件包内的文件信息；
－qpi：列出RPM软件包的描述信息；
－qf：查找指定文件属于哪个RPM软件包；
－Va：校验所有的RPM软件包，查找丢失的文件；
－qa: 查找相应文件，如 rpm -qa mysql



## RPM的打包


## RPM的spec语法

## RPM解包获取文件


