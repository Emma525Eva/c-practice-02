# ls note 02

#### 1.`opendir()` + `readdir()`

作用：**打开一个目录，然后逐个读取目录里面的文件和子目录。**

- `opendir()`：打开一个目录，获得目录流指针。
- `readdir()`：从目录流中读取下一个目录项。
- `closedir()`：关闭目录流，释放相关资源。

1.`opendir()`--打开目录

（1）头文件

```c
#include <dirent.h>
```

（2）函数原型

```c
DIR *opendir(const char *name);
```

| 部分               | 含义               |
| ------------------ | ------------------ |
| `DIR *`            | 返回一个目录流指针 |
| `opendir`          | 函数名称           |
| `const char *name` | 要打开的目录路径   |

2.`readdir()`

函数原型

```c
struct dirent *readdir(DIR *dirp);
```

| 部分            | 含义                     |
| --------------- | ------------------------ |
| struct dirent * | 返回一个目录项结构体指针 |
| readdir         | 函数名称                 |
| DIR *dirp       | 已经打开的目录流指针     |

3.`closedir()`

（1）函数原型

```c
int closedir(DIR *dirp);
```

（2）返回值：

- 成功：返回 `0`。
- 失败：返回 `-1`。

（3）为什么需要关闭

因为打开目录会占用系统资源。使用结束后应该及时释放，避免资源泄漏。

#### 2.思路

**第一阶段：参数解析**

开始

使用 `getopt()`，实现七个参数的识别与组合。

**第二阶段：目录读取**

使用 `opendir()`、`readdir()`，获取文件名，实现 `-a`。

**第三阶段：文件信息**

使用 `lstat()`，设计 `FileInfo` 结构体，保存文件属性。

**第四阶段：格式化输出**

实现 `-l`、`-i`、`-s`，包括权限、用户、组、时间等。

**第五阶段：排序**

实现 `-t`、`-r`。

**第六阶段：递归**

实现 `-R`，测试 `/`。

**第七阶段：界面美化**

文件颜色、目录颜色、输出对齐。

**第八阶段：综合测试**

检查内存泄漏、资源释放、异常情况和各种参数组合。

#### 3.在 Linux 中，通常以 `.` 开头的文件名属于隐藏文件。

#### 4.DIR

`DIR` 是一个目录流类型，用来表示一个已经打开的目录及其读取状态。

可以暂时这样理解：

- 普通文件操作有 `FILE *`。
- 目录操作有 `DIR *`。

```c
DIR *dir;
```

**声明一个名为 dir 的指针变量，它将来可以保存一个 DIR 类型对象的地址。**

注意：`opendir()` 函数返回的是 `DIR *`

```c
dir = opendir(path);
```

这行代码实际上完成了两个动作：

1. 调用 `opendir()` 打开目录。
2. 把返回的目录流指针保存到 `dir` 中。

假设你的函数是：

```c
void list_directory(const char *path, const Options *options)
```

这里：

```c
const char *path
```

表示接收一个目录路径字符串。

例如调用：

```c
list_directory("/home/emma", &options);
```

那么函数里面的 `path` 就代表：

```c
/home/emma
```

#### 5.struct dirent *entry;

`dirent` 是结构体的名称。

它不是 C 语言的关键字，而是 Linux/POSIX 目录操作接口提供的结构体类型名称。

需要包含头文件：

```c
#include <dirent.h>
```

可以把 `struct dirent` 理解成一个专门用来描述目录项信息的结构体。

它有一些成员，其中我们最常使用的是：

```c
d_name
```

`d_name` 保存目录项的名称。

这里的 `*` 表示：

`entry` 是一个指针变量，它可以保存一个 `struct dirent` 结构体的地址。

```
entry->d_name
```

这里的 `->` 是 C 语言专门用于**通过结构体指针访问结构体成员**的运算符。

它等价于：

```c
(*entry).d_name
```

含义就是：

**通过** `entry` **指针找到当前的** `struct dirent` **结构体，再访问其中的** `d_name` **成员。**

而 `d_name` 保存的就是当前目录项的名称。

#### 6.while((entry=readdir(dir))!=NULL)

```c
readdir(dir)
```

这是一个函数调用。

它的作用是：

从 `dir` 所表示的目录流中读取下一个目录项。

这里的 `dir` 是之前通过 `opendir()` 获得的目录流指针：

```c
DIR *dir;
dir = opendir(path);
```

```c
entry = readdir(dir);
```

这条语句的意思是：

1. 调用 `readdir(dir)`。
2. 读取下一个目录项。
3. 获得一个指向目录项结构体的指针。
4. 将这个指针保存到 `entry` 中。

#### 7.errno

`errno` 是 C 语言错误处理机制中使用的一个错误码变量。

它通常用于记录某些系统调用或库函数发生错误时的原因。

例如：

- 打开文件失败；
- 打开目录失败；
- 读取目录失败；
- 文件操作失败。

使用它需要包含头文件：

```c
#include <errno.h>
```

注意：`errno` 并不是一个专门表示程序是否出错的布尔变量，而是一个错误码。

例如：

```
errno = 0;
```

表示把错误码清零。

如果某个函数发生错误，可能会设置 `errno` 为一个非零值。

常见错误码包括：

| 错误码   | 含义             |
| -------- | ---------------- |
| `ENOENT` | 文件或目录不存在 |
| `EACCES` | 没有访问权限     |
| `EBADF`  | 文件描述符无效   |
| `ENOMEM` | 内存不足         |

这些错误码通常也需要通过：

```c
#include <errno.h>
```

来使用。

#### 8.struct stat file_info;

这是一个结构体变量声明。

`struct stat` 是 Linux 系统提供的一个结构体类型。它用来保存文件或目录的属性信息。

需要包含头文件：

```c
#include <sys/stat.h>
```

它可以保存很多信息，例如：

| 成员       | 含义                |
| ---------- | ------------------- |
| `st_mode`  | 文件类型和权限      |
| `st_size`  | 文件大小            |
| `st_ino`   | inode 编号          |
| `st_uid`   | 文件所有者的用户 ID |
| `st_gid`   | 文件所属组的组 ID   |
| `st_mtime` | 最后修改时间        |
| `st_nlink` | 硬链接数量          |

```c
file_info
```

这是变量名称，可以自己命名。

#### 9.if (stat(path, &file_info) == -1)

`stat()` 是 Linux/POSIX 提供的文件属性查询函数。

需要包含：

```c
#include <sys/stat.h>
```

函数原型：

```c
int stat(const char *pathname, struct stat *statbuf);
```

| 内容                   | 含义                         |
| ---------------------- | ---------------------------- |
| `int`                  | 返回整数                     |
| `stat`                 | 函数名称                     |
| `const char *pathname` | 要查询的路径                 |
| `struct stat *statbuf` | 用来接收文件属性的结构体指针 |

它的作用是：

**根据给定的路径，获取对应文件或目录的属性信息。**

例如：

```c
stat("/home/emma", &file_info);
```

就是查询 `/home/emma` 对应的文件属性，并将结果保存到 `file_info` 中。

`stat()` 的返回值：

- 返回 `0`：获取属性成功。
- 返回 `-1`：获取属性失败。

#### 10.if (S_ISDIR(file_info.st_mode))

它的作用是：**判断当前路径对应的对象是不是目录。**

```c
struct stat file_info;
```

其中有一个成员：

```c
st_mode
```

它保存文件类型和权限信息。

例如：

```c
file_info.st_mode
```

可以获得文件的类型和权限位。

注意：`st_mode` 不只是一个简单的文件类型编号，它还包含权限信息。

```c
S_ISDIR()
```

这是系统提供的一个宏，用于判断文件类型是不是目录。

需要包含：

```c
#include <sys/stat.h>
```

它会检查 `st_mode` 中的文件类型信息。

结果：

- 是目录：条件成立。
- 不是目录：条件不成立。

如果它是目录，那么：

```c
S_ISDIR(file_info.st_mode)
```

就会得到真。

#### 11.int main(int argc, char *argv[])

在普通 C 程序中，程序从 `main()` 开始执行。

`argc` 是一个整数参数。它表示：

**用户在命令行中输入的参数数量。**

例如，你执行：

```c
./myls -a /home/emma
```

程序收到的参数大致是：

| 参数位置  | 内容         |
| --------- | ------------ |
| `argv[0]` | `./myls`     |
| `argv[1]` | `-a`         |
| `argv[2]` | `/home/emma` |

一共三个参数。

因此：

```c
argc == 3
```

注意：`argc` 统计的是包括程序名称在内的参数数量。

```
char *argv[]
```

这是一个字符指针数组参数。它用来保存用户在命令行中输入的各个参数字符串。

#### 12.if (optind == argc)

这是整个 `main()` 中非常重要的一行。

它负责判断：**用户有没有指定要查看的文件或目录路径。**

`optind` 是 `getopt()` 使用的一个全局变量。

它需要包含头文件：

```
#include <unistd.h>
```

它的作用是：**记录下一个尚未处理的命令行参数在** `argv` **中的下标。**

例如：

```c
./myls -a /home/emma
```

`getopt()` 解析完 `-a` 后：

```c
optind == 2
```

因为：

```c
argv[0] = "./myls";
argv[1] = "-a";
argv[2] = "/home/emma";
```

下一个未处理的参数是：

```c
argv[2]
```

它就是用户输入的路径。

再看：

```c
./myls
```

没有任何选项，也没有指定路径。

这时：

```c
optind == 1
argc == 1
```

因此：

```c
optind == argc
```

条件成立。

**为什么用** `optind == argc`**？**

因为：

- `argc`：参数总数量。
- `optind`：下一个未处理参数的位置。

如果：

```c
optind == argc
```

说明已经没有剩余参数了。也就是说，用户没有提供路径。

#### 13.list_path(".", &options);

```c
if(optind==argc)
      list_path(".",&options);
```

表示：**如果用户没有指定路径，就默认读取当前目录。**

```c
"."
```

**这是一个字符串，表示当前工作目录。**

Linux 中常见的路径符号：

| 符号 | 含义                              |
| ---- | --------------------------------- |
| `.`  | 当前目录                          |
| `..` | 上一级目录                        |
| `/`  | 根目录                            |
| `~`  | 当前用户的家目录（由 Shell 展开） |

#### 14.功能测试宏（Feature Test Macros）

添加的是：

```c
#define _DEFAULT_SOURCE
#define _POSIX_C_SOURCE 200809L
```

它们都叫作**功能测试宏（Feature Test Macros）**。

在程序中使用了：

```c
#include <unistd.h>
```

然后调用：

```c
getopt(argc, argv, "alRtris");
```

以及：

```c
if (optind == argc)
```

这里有一个问题：

`unistd.h` 并不是把所有系统接口无条件地提供给程序。

在 Linux 中，系统头文件会根据你设置的功能测试宏，决定向程序暴露哪些接口声明。

例如：

```c
#define _POSIX_C_SOURCE 200809L
#include <unistd.h>
```

相当于告诉系统：

> 我的程序希望使用 POSIX.1-2008 标准规定的接口。

如果没有设置合适的宏，某些接口的声明可能不会出现在头文件中。

这样编译器就可能出现：

```
error: implicit declaration of function ‘getopt’
error: ‘optind’ undeclared
```

注意：这不是说 `getopt()` 不存在，而是编译器没有看到它的声明。

1.

```
#define _POSIX_C_SOURCE 200809L
```

它的作用是：**指定程序希望使用哪个版本的 POSIX 标准接口。**

POSIX 是一套操作系统接口标准。

它规定了很多常用的系统功能，例如：

- 文件操作
- 进程管理
- 目录操作
- 输入输出
- 命令行参数处理

Linux 提供了大量 POSIX 接口。

例如：

```c
#include <unistd.h>
```

其中就有一些 POSIX 接口：

```c
read();
write();
close();
```

**为什么是** `200809L`**？**

它代表 POSIX.1-2008 对应的标准版本。

常见值：

| 宏值      | 对应标准      |
| --------- | ------------- |
| `199309L` | POSIX.1b-1993 |
| `199506L` | POSIX.1c-1995 |
| `200112L` | POSIX.1-2001  |
| `200809L` | POSIX.1-2008  |

这里的 `L` 表示整数常量使用 `long int` 类型。

可以把：

```
#define _POSIX_C_SOURCE 200809L
```

理解为：请按照 POSIX.1-2008 的接口标准，向我的程序提供相应的声明。

2.

```c
#define _DEFAULT_SOURCE
```

它的作用是：**启用 glibc 提供的默认扩展接口。**

glibc 是 GNU C Library，也就是 GNU C 标准库实现。

Ubuntu 上的 GCC 通常使用 glibc 提供的 C 库和系统头文件。

glibc 除了提供标准 C 和 POSIX 接口，还提供一些 GNU/Linux 环境下常用的扩展接口。

例如：

```c
strdup();
```

`strdup()` 是 POSIX 接口，同时也被 glibc 提供。

再比如：

```c
getline();
```

它也由 glibc 提供，并属于 POSIX.1-2008。

`_DEFAULT_SOURCE` 主要用于启用 glibc 的默认接口集合。

可以理解为：除了标准规定的接口，我还希望使用 glibc 默认提供的一些扩展接口。
