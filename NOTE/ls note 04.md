# ls note 04

### 1.FileInfo **files

1.定义了一个结构体：

```c
typedef struct
{
    char name[256];      // 文件名
    long size;           // 文件大小
    long inode;          // inode 编号
} FileInfo;
```

它表示一个文件的信息。

2.

```c
FileInfo file;
```

这表示：

定义一个 `FileInfo` 类型的变量。

它直接保存一个结构体。

例如：

```c
FileInfo file;

file.size = 1024;
file.inode = 1001;
```

3.

```c
FileInfo *files;
```

这里多了一个 `*`。

它表示：

**files 是一个指针，指向 FileInfo 类型的对象。**例如：

```c
FileInfo *files = NULL;

files = malloc(3 * sizeof(FileInfo));
```

定义一个指针，暂时不指向任何有效的结构体。

向堆内存申请可以容纳 3 个 `FileInfo` 结构体的空间。

把申请到的内存地址交给 `files`。

此时，`files` 就指向动态数组的第一个元素。

4.

```c
FileInfo **files;
```

这里有两个 `*`。

它表示：**files 是一个指针，它指向另一个指针，而另一个指针指向 FileInfo。**

也就是说：

- `FileInfo`：结构体。
- `FileInfo *`：指向结构体的指针。
- `FileInfo **`：指向结构体指针的指针。

5.**为什么函数需要 FileInfo \**files？**

先看调用函数之前的代码：

```c
FileInfo *files = NULL;
```

此时：

```c
files = NULL
```

假设需要写一个函数，把文件信息添加到动态数组中：

```c
int add_file_info(FileInfo **files, ...)
```

为什么不直接写成：

```c
int add_file_info(FileInfo *files, ...)
```

原因在于：

**C 语言函数参数默认是值传递。**

通过一个例子理解。

假设：

```c
void change_pointer(FileInfo *files)
{
    files = malloc(sizeof(FileInfo));
}
```

调用：

```c
FileInfo *files = NULL;

change_pointer(files);
```

你可能认为：函数里面给 `files` 分配了内存，外面的 `files` 也应该改变。

但实际上不是。

为什么？

因为调用函数时，C 语言会把外面的指针值复制一份给函数参数。

现在函数执行：

```c
files = malloc(sizeof(FileInfo));
```

修改的是函数内部的指针副本。

外面的 `files` 仍然是 `NULL`。

函数结束后，外面的指针没有得到新地址。

甚至这块申请的内存也无法通过外面的指针访问，造成内存泄漏。

改成：

```c
void change_pointer(FileInfo **files)
{
    *files = malloc(sizeof(FileInfo));
}
```

调用：

```c
FileInfo *files = NULL;

change_pointer(&files);
```

为什么传入 `&files`？

因为：

```c
files
```

表示指针变量本身。

而：

```c
&files
```

表示这个指针变量在内存中的地址。

函数参数：

```c
FileInfo **files
```

正好可以接收这个地址。

现在函数内部：

```c
*files = malloc(sizeof(FileInfo));
```

这里的 `*files` 就是调用者的原始指针变量。

因此，修改：

```c
*files = ...
```

实际上就是修改外面的：

```c
FileInfo *files
```