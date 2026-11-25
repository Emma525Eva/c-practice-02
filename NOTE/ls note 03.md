# ls note 03

### 1.  ./

1.`./` 表示：**当前目录下的**。

在 Linux 中，`./` 是一个相对路径写法。

在 Linux 中：

| 符号 | 含义       |
| ---- | ---------- |
| `.`  | 当前目录   |
| `..` | 上一级目录 |
| `/`  | 根目录     |

2.当你输入：

```c
./myls
```

可以拆成：

```
./       myls
当前目录  可执行文件
```

完整意思就是：**运行当前目录下的 myls 程序。**

### 2.struct stat

```c
struct stat file_info;
lstat("main.c", &file_info);
```

1. `struct stat`：结构体类型。
2. `file_info`：结构体变量。
3. `lstat()`：获取文件属性的函数。

1.

Linux 系统提供了很多结构体，`struct stat` 就是其中之一，它专门用来保存文件的属性信息。

2.

```c
struct stat file_info;
```

| 部分        | 含义           |
| ----------- | -------------- |
| `struct`    | 声明结构体类型 |
| `stat`      | 结构体的名称   |
| `file_info` | 变量名称       |
| `;`         | 语句结束       |

它的意思是：**创建一个名为** `file_info` **的变量，用来保存文件属性。**

注意：

`struct stat` 是 Linux 系统提供的类型，不是你自己定义的。

要使用它，需要包含：

```c
#include <sys/stat.h>
```

3.

`lstat()` 是 Linux/POSIX 提供的系统接口，用于获取文件属性。

它的函数原型是：

```c
int lstat(const char *pathname, struct stat *statbuf);
```

需要的头文件：

```c
#include <sys/stat.h>
#include <unistd.h>
```

**第一个参数：**

```c
const char *pathname
```

表示文件路径。

例如：

```
lstat("main.c", &file_info);
```

这里 `"main.c"` 就是要查询的文件。

它也可以是完整路径：

```c
lstat("/home/emma/main.c", &file_info);
```

**第二个参数：**

```c
struct stat *statbuf
```

这是一个结构体指针。

它的作用是：告诉 `lstat()` 把获取到的文件信息保存在哪里。

例如：

```
struct stat file_info;

lstat("main.c", &file_info);
```

这里：

```
&file_info
```

表示获取 `file_info` 变量的地址。

为什么需要地址？

因为 `lstat()` 要修改 `file_info` 中的数据。

如果只传入：

```
file_info
```

传递的就是结构体变量本身，而不是我们需要的结构体地址。

**返回值：**

| 返回值 | 含义     |
| ------ | -------- |
| `0`    | 获取成功 |
| `-1`   | 获取失败 |

4.

执行成功后，文件属性就保存在 `file_info` 中。

那么如何读取这些信息？

使用：

```
file_info.成员名称
```

例如：

```
file_info.st_size
```

表示文件大小。

如果拿到的是结构体指针，例如：

```
struct stat *p;
```

那么使用：

```
p->st_size
```

5.`st_mode`**：文件类型和权限**

```
file_info.st_mode
```

它保存两类信息：

- 文件类型。
- 文件访问权限。

例如，Linux 文件类型有：

| 类型     | 含义               |
| -------- | ------------------ |
| 普通文件 | `.c`、`.txt` 等    |
| 目录     | 文件夹             |
| 符号链接 | 指向其他文件的链接 |
| 字符设备 | 字符设备文件       |
| 块设备   | 块设备文件         |
| FIFO     | 命名管道           |
| Socket   | 套接字文件         |

那么，如何判断文件类型？

系统提供了很多宏。

例如：

```c
S_ISDIR(file_info.st_mode)
```

判断是不是目录。

```c
S_ISREG(file_info.st_mode)c
```

判断是不是普通文件。

```c
S_ISLNK(file_info.st_mode)
```

判断是不是符号链接。

它们都需要：

```c
#include <sys/stat.h>
```

注意：这些宏返回非零值表示条件成立，返回 0 表示不成立。

6.`st_ino`**：inode 编号**

```
file_info.st_ino
```

它表示文件的 inode 编号。

在 Linux 文件系统中，文件不仅有文件名，还有一个用于标识文件的 inode。

可以简单理解为：

- 文件名：方便用户识别文件。
- inode：文件系统用来标识文件的编号。

7.`st_size`**：文件大小**

```
file_info.st_size
```

表示文件大小，单位是字节（Byte）。

8.`st_nlink`**：硬链接数量**

```
file_info.st_nlink
```

表示文件的硬链接数量。

假设有一个文件：

```
main.c
```

执行：

```
ln main.c copy.c
```

就创建了一个硬链接 `copy.c`。

此时：

```
main.c ──┐
         ├── 同一个 inode
copy.c ──┘
```

两个文件名指向同一个 inode,因此硬链接数量通常会增加。

可以查看：

```
ls -l main.c copy.c
```

输出中会有类似：

```
-rw-r--r-- 2 emma emma 1000 Nov 25 20:00 main.c
-rw-r--r-- 2 emma emma 1000 Nov 25 20:00 copy.c
```

其中的 `2` 就是硬链接数量。

9.`st_uid`**：文件所有者 ID**

```
file_info.st_uid
```

表示文件所有者的用户 ID。

Linux 中每个用户都有一个 UID。例如，可以执行：

```
id
```

查看当前用户的 UID。

如果文件属于当前用户，那么 `st_uid` 通常就等于当前用户的 UID

10.`st_gid`**：文件所属组 ID**

```
file_info.st_gid
```

表示文件所属用户组的 ID。

Linux 中用户可以属于一个或多个组。

11.`st_mtime`**：最后修改时间**

```
file_info.st_mtime
```

表示文件内容最后一次修改的时间。它的类型是 `time_t`。

如果想把它转换成人类容易阅读的时间，可以使用：

```c
#include <time.h>
```

例如：

```c
char *time_str = ctime(&file_info.st_mtime);
printf("修改时间：%s", time_str);
```

`ctime()` 会将时间转换为可读字符串。

例如：

```c
修改时间：Wed Nov 25 20:00:00 2026
```

注意：`ctime()` 返回的字符串通常自带换行符，所以这里不需要再额外添加 `\n`。

12.`st_blocks`**：文件占用的磁盘块数**

```
file_info.st_blocks
```

表示文件实际占用的磁盘块数量。

这里要区分两个概念：

- `st_size`：文件逻辑大小。
- `st_blocks`：文件实际分配的磁盘块数量。

假设一个文件：

```
st_size = 1000 字节
st_blocks = 8
```

在 Linux 常见文件系统中，`st_blocks` 通常以 512 字节为单位，因此实际分配空间约为：\[ 8 \times 512 = 4096\text{ 字节} \]

注意：这两个数不一定成正比。稀疏文件等特殊情况可能出现 `st_blocks` 对应的空间小于 `st_size`。

### 3.符号链接

符号链接可以理解为 Linux 中的快捷方式。

例如，有：

```
main.c
```

然后执行：

```
ln -s main.c link.c
```

就创建了一个符号链接：

```
link.c → main.c
```

可以用下面的命令查看：

```
ls -l
```

你可能看到：

```
-rw-r--r-- 1 emma emma 1000 main.c
lrwxrwxrwx 1 emma emma    6 link.c -> main.c
```

这里：

```
link.c -> main.c
```

表示 `link.c` 指向 `main.c`。

### 4.`lstat()` 和 `stat()`区别

先看两个函数的原型：

```c
int stat(const char *pathname, struct stat *statbuf);
int lstat(const char *pathname, struct stat *statbuf);
```

它们的参数和返回值基本相同。

**真正的区别在于：遇到符号链接时，它们处理的对象不同。**

1.`stat()`**：获取目标文件的属性**

假设：

```
link.c → main.c
```

执行：

```
struct stat file_info;
stat("link.c", &file_info);
```

`stat()` 会跟随符号链接，获取它指向的 `main.c` 的属性

所以：

```c
S_ISREG(file_info.st_mode)
```

通常会返回真，因为 `main.c` 是普通文件。

2.`lstat()`**：获取符号链接本身的属性**

```c
struct stat file_info;
lstat("link.c", &file_info);
```

`lstat()` 不会跟随符号链接。

它直接获取 `link.c` 本身的属性。

所以：

```c
S_ISLNK(file_info.st_mode)
```

会返回真。