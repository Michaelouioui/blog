# Linux程序开发环境

## CH 1 概述和安装

### 推荐购买的书

Linux程序设计《人民邮电出版社》

深入理解LINUX内核《中国电力出版社》

### Linux系统的特点

- 与Unix操作系统兼容
- 广泛的硬件运行环境
- 强大的网络能力
- 对多用户多任务的完美支持
- 多硬件平台支持和可移植性
- 良好的设备独立性

### CentOS

建议使用CentOS7

### Ubuntu

很漂亮 

建议使用LTS版本

### Linux系统界面

- 图形化用户界面：GNOME、KDE
- 命令行用户界面CLI
- CLI模拟器界面

### 一些题外话

对部署在服务器的终端一般采用完全终端界面进行操作  

学习的时候可以使用图形化界面

### 命令语法结构

$ command [[-]option(s)] [option argument(s)] [command argument(s)]

- command是Linux命令(不同Shell有所区别)大小写严格区分，一般为小写
- [-option(s)]是命令的一个或多个选项，可以分为唱歌时和短格式，一般为短格式
- [option argument(s)]是选项动作的一个或多个参数
- [command argument(s)]受命令影响的一个或多个对象

### 例子

- ![image-20210207151547290](C:\Users\86135\AppData\Roaming\Typora\typora-user-images\image-20210207151547290.png)
- ![image-20210207151622883](C:\Users\86135\AppData\Roaming\Typora\typora-user-images\image-20210207151622883.png)
- ![image-20210207151634866](C:\Users\86135\AppData\Roaming\Typora\typora-user-images\image-20210207151634866.png)

### 常用技巧

- 命令历史：方向键上下
- 自动补全：Tab
- 清屏：clear命令
- 退出命令行界面：exit命令
- 查看手册：man命令

## CH 2 用户与文件基本操作

### Linux文件系统

一切事物都是文件

Linux文件系统是一个典型的单根、分层、树形结构，第一级目录为根目录，由“/”来表示

其他特点

- 文件名严格区分大小写，支持空格、“.”、“-”、“_”
- “.“开头的文件名为隐藏文件，默认不显示
- 目录（文件夹）也是文件，是一种属性不同的文件
- 后缀名是不是文件名称的必须要素，仅用于用户方便使用

### 常用目录说明

![image-20210207214110014](C:\Users\86135\AppData\Roaming\Typora\typora-user-images\image-20210207214110014.png)

![image-20210207214148961](C:\Users\86135\AppData\Roaming\Typora\typora-user-images\image-20210207214148961.png)

 ![image-20210207214837717](C:\Users\86135\AppData\Roaming\Typora\typora-user-images\image-20210207214837717.png)

![image-20210207214845671](C:\Users\86135\AppData\Roaming\Typora\typora-user-images\image-20210207214845671.png)

### 文件相关操作

#### pwd : Print Working Directory

pwd [-P]

- -P : 代表当前目录显示当前路径，而非使用链接路径

![image-20210207215310553](C:\Users\86135\AppData\Roaming\Typora\typora-user-images\image-20210207215310553.png)

#### which

获取命令的可执行文件位置

which [-a]

- -a : 如果有多个可执行文件，列出所有的可执行文件位置

![image-20210207220131190](C:\Users\86135\AppData\Roaming\Typora\typora-user-images\image-20210207220131190.png)

#### ls : list

ls [-aAdFhlrSt]

![image-20210207220356664](C:\Users\86135\AppData\Roaming\Typora\typora-user-images\image-20210207220356664.png)

#### cd : Change Directory

![image-20210207220731598](C:\Users\86135\AppData\Roaming\Typora\typora-user-images\image-20210207220731598.png)

#### du : Disk Usage

du [-aAdFhlrSt]

- -h : human-readable,以K、G、M的方式来显示大小
- -a : 默认只显示目录大小，-a显示文件和目录的大小
- -s : 显示总计大小

![image-20210208095936454](C:\Users\86135\AppData\Roaming\Typora\typora-user-images\image-20210208095936454.png)

#### cat : concatenate files and print on the standard output

一次性显示文件内容（整个文件打开）

cat 文件名

![image-20210208100156591](C:\Users\86135\AppData\Roaming\Typora\typora-user-images\image-20210208100156591.png)

#### less

分页显示文件内容

![image-20210208100436954](C:\Users\86135\AppData\Roaming\Typora\typora-user-images\image-20210208100436954.png)

#### 其他显示文件内容的命令

![image-20210208100623455](C:\Users\86135\AppData\Roaming\Typora\typora-user-images\image-20210208100623455.png)

#### od

显示非纯文本文件内容

od [-t TYPE]

- -t : 指定输出内容的类型
- a : 默认字符
- c : 使用ASCII字符输出
- dox : 十进制、八进制、十六进制
- f : 浮点数输出

![image-20210208100857332](C:\Users\86135\AppData\Roaming\Typora\typora-user-images\image-20210208100857332.png)

#### touch

新建文件（本意是修改文件的最后修改时间，如果文件不存在就会新建文件）

![image-20210208101219174](C:\Users\86135\AppData\Roaming\Typora\typora-user-images\image-20210208101219174.png)

#### mkdir : Make Directory

mkdir [-mp] 目录名称

- -m : 创建目录是指定权限，并且时直接设置，不考虑默认权限
- -p : 将所需的目录递归创建

![image-20210208101751043](C:\Users\86135\AppData\Roaming\Typora\typora-user-images\image-20210208101751043.png)

#### cp : Copy

cp [-adffilprsu] 源文件 目标文件

cp [options] 源文件1 源文件2 ... 目标文件

- -a :相当于-pdr的意思 
- -d : 若源文件为链接文件，则复制链接文件而不是文件本身
- -i : 若目标文件已存在，则覆盖时会询问交互
- -p : 联通文件属性一起复制，而不是
- -r : 递归持续复制

如果直接复制，只会复制文件的目录，因此需要递归复制，一般使用-a

![image-20210208102234221](C:\Users\86135\AppData\Roaming\Typora\typora-user-images\image-20210208102234221.png)

#### rm : Remove

rm [-fir] 文件或目标

- -f : 强制删除，忽略不存在文件，不出现警告信息
- -i : 互动模式，删除前会询问
- -r : 递归删除

![image-20210208102600051](C:\Users\86135\AppData\Roaming\Typora\typora-user-images\image-20210208102600051.png)

### 文件的三个时间属性

- mtime : 当该文件的内容**更改**时就会修改该时
- ctime : 当该文件属性（状态）**更改**时就会修改该时间
- atime : 当该文件的内容被**访问**时就会修改该时间

### Linux用户和用户组

#### 基本概念

- Linux系统中的文件、进程（文件）都和用户的概念绑定在一起
- 根据与文件的关系将用户分为所有者（User）和使用者，为了更加清晰地规定使用权限，又将使用者区分组内用户（Group）和其他用户（Other）
- Linux系统中地用户必须隶属于一个或多个组
- 系统分别针对这三种用户桂东能进行二点操作和行为

#### 用户相关文件

- 用户账号信息：/etc/passwd
- 用户密码信息：/etc/shadow
- 组账号信息：/etc/group
- 组管理及密码信息：/etc/gshadow 

#### /etc/passwd

![image-20210208103436617](C:\Users\86135\AppData\Roaming\Typora\typora-user-images\image-20210208103436617.png)

账号信息：密码：UID：GID：用户信息：主文件夹：Shell

每一行代表一个账号，除了用户账号还有一些系统账号

#### /etc/shadow

![image-20210208103641589](C:\Users\86135\AppData\Roaming\Typora\typora-user-images\image-20210208103641589.png)

#### /etc/group

![image-20210208103707009](C:\Users\86135\AppData\Roaming\Typora\typora-user-images\image-20210208103707009.png)

![image-20210208103734294](C:\Users\86135\AppData\Roaming\Typora\typora-user-images\image-20210208103734294.png)

### 账号管理

#### useradd : 新增账号

![image-20210208103839648](C:\Users\86135\AppData\Roaming\Typora\typora-user-images\image-20210208103839648.png)

#### passwd : 密码管理

![image-20210208103901439](C:\Users\86135\AppData\Roaming\Typora\typora-user-images\image-20210208103901439.png)

####  其他常用命令

![image-20210208104018046](C:\Users\86135\AppData\Roaming\Typora\typora-user-images\image-20210208104018046.png)

### 文件权限

![image-20210208104631255](C:\Users\86135\AppData\Roaming\Typora\typora-user-images\image-20210208104631255.png)

![image-20210208104613874](C:\Users\86135\AppData\Roaming\Typora\typora-user-images\image-20210208104613874.png)

#### 权限字符串：-rwxrwxrwx

r : 表示具有读取目录的权限，即可以用ls命令查询该目录下的文件信息

w : 表示具有更改该目录结构猎鸟的权限。包含：新建新的文件（目录）、删除已经存在的文件与目录、将已存在的文件（目录）进行重命名、转移该目录内的文件（目录）的文职

x : 表示是否能进入目录

#### 文件类型

##### 权限字符串地第一个字符表示文件类型

- d : 目录
- "-" : 文件
- l : 链接文件
- b : 面向块地设备文件，一般时供储存地接口设备
- c : 面向字符的设备文件，即串行设备，例如键盘、鼠标

##### file : 确定文件的具体类型

![image-20210208110200107](C:\Users\86135\AppData\Roaming\Typora\typora-user-images\image-20210208110200107.png)

#### Linux使用UGO模型来进行权限管理

- U代表文件所有者
- G代表所有者所在组成员
- O代表其他用户
- 每个文件用9个权限字符来表示，3个一组，分别代表UGO的读、写、执行权限，如rwxrw-rw-，-表示不具备该权限，字符则表示有该权限

![image-20210208110453021](C:\Users\86135\AppData\Roaming\Typora\typora-user-images\image-20210208110453021.png)

#### chmod : 改变/设置文件权限

chmod [-R] 权限表达式 文件名

权限表达式可以有字符和八进制数字两种表达方式

![image-20210208110737373](C:\Users\86135\AppData\Roaming\Typora\typora-user-images\image-20210208110737373.png)

##### 两种权限修改方式

- 使用数字时直接在chmod命令后跟数字形式的权限字符串

- 使用字符时用下面方式操作

   ![image-20210208110957301](C:\Users\86135\AppData\Roaming\Typora\typora-user-images\image-20210208110957301.png)

 ![image-20210208111211388](C:\Users\86135\AppData\Roaming\Typora\typora-user-images\image-20210208111211388.png)

![image-20210208111225092](C:\Users\86135\AppData\Roaming\Typora\typora-user-images\image-20210208111225092.png)

#### 改变文件（目录）所有属性的命令chown、chgrp

![image-20210217114033686](C:\Users\86135\AppData\Roaming\Typora\typora-user-images\image-20210217114033686.png)

### 杂

如果在不同的地方存有相同的文件，系统会按照环境变量的顺序来进行查找，执行找到的第一个文件

-h一般表示用人类可以看懂的形式来进行表示

默认权限：umask

新增用户之后密码若为空则不能登录

第一次登录之后必须要改密码

## CH 3 高级文件操作和磁盘

### 文件操作进阶

#### 搜索与查找

##### which：命令查找

![image-20210217115943935](C:\Users\86135\AppData\Roaming\Typora\typora-user-images\image-20210217115943935.png)

##### whereis：范围内查找

![image-20210217120022522](C:\Users\86135\AppData\Roaming\Typora\typora-user-images\image-20210217120022522.png)

##### locate：索引查询

![image-20210217120055212](C:\Users\86135\AppData\Roaming\Typora\typora-user-images\image-20210217120055212.png)





#### $PATH : 环境变量

- 环境变量用$表示，一般大写
- $PATH变量用于保存可执行程序的路径
- SHELL会在$PATH自左向右一次搜寻对应的程序名

![image-20210217121921269](C:\Users\86135\AppData\Roaming\Typora\typora-user-images\image-20210217121921269.png)

















## CH 4 数据流、重定向与进程

## CH 5 Shell脚本编程

## CH 6 内存、磁盘与文件

## CH 7 进程与服务

## CH 8 网络与安全

## CH 9 Java Web开发环境（上）

## CH 10 Java Web开发环境（下）

## CH 11 高并发服务器环境

## CH 12 高可用服务器环境







