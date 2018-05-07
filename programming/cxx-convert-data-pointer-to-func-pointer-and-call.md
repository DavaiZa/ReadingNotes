# 实验1

把一个函数地址当做普通指针memcpy到一段堆内存，然后把这段堆内存的首地址当做函数指针来调用。代码如下

```C
/// test.c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

typedef void(*pfunc)();
void* buf;

void foo()
{
	printf("The man who changed China!");
}

int main()
{
	buf = malloc(1024);
	memcpy(buf, &foo, 128);
	pfunc foo_ = buf;
	foo_();
}
```

写到这里时，由于我还没有学会如何获取函数体的长度，所以姑且认为foo的长度为128。

结果：在`foo_()`处发生段错误。原因很简单，malloc出来的堆内存显然是不可执行的。

这里稍微跑个题，为什么C99标准要区分数据指针和函数指针呢？因为C语言应该能在两种架构（冯·纽曼、哈佛）的计算机上使用。而哈佛架构的内存分为代码内存和数据内存两个不同的部分——用代码内存的地址来寻址数据内存没有任何意义，反之亦然。关于这两种架构，详见https://www.zhihu.com/question/22406681。

即使现今大部分计算机的指令和数据使用相同的地址空间，也需要由操作系统对内存进行分段。这种分段不仅分出堆、栈、代码、常量等内存区域，也标志了这些区域的访问权限。

# 那么，怎样让堆内存变得可执行？

我在stackoverflow上搜到了这样一个问题，[What do I have to do to execute code in data areas, ( segment protection )](https://stackoverflow.com/questions/28015876/what-do-i-have-to-do-to-execute-code-in-data-areas-segment-protection)。[其中一个回答](https://stackoverflow.com/a/28016270)详细地讲解了如何使用 `__attribute__((section))`、`mprotect`和`mmap` 三种不同的手段来执行堆内存中的代码。

## 使用mprotect（捎带`__attribute__((section))`）

mprotect来自头文件`<sys/mman.h>` ，它能重新标记一个虚拟内存页的访问权限。详见http://man7.org/linux/man-pages/man2/mprotect.2.html

传入mprotect的地址必须对齐到页边界。为了申请一个这样的内存，还需要用到[`aligned_alloc`](http://en.cppreference.com/w/c/memory/aligned_alloc)和[`sysconf`](http://man7.org/linux/man-pages/man3/sysconf.3.html)。

```C
// compile with --std=c99

#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/mman.h>
#include <stdint.h>

typedef int(*pfunc)();

/* 把机器码写在自己声明的内存段.my_executable_blob上。这个段拥有读、写、执行权限 */
__attribute__((section(".my_executable_blob,\"awx\",@progbits#")))
static uint8_t code[] = {
	0xB8,0x2A,0x00,0x00,0x00,   /* mov  eax, 42     */
	0xC3,                       /* ret              */
};

int main()
{
	pfunc func = (void*)code;
	printf("(static) code returned %d\n", func());
	int PAGESIZE = sysconf(_SC_PAGESIZE);
	void* buf = aligned_alloc(PAGESIZE, sizeof(code));
	memcpy(buf, code, sizeof(code));
	mprotect(buf, sizeof(code), PROT_EXEC); 
	func = buf;
	printf("(dynamic) code returned %d\n", func());
}
```

据答案所说，mprotect可能会被selinux禁掉。目前我还不知道selinux是否能单独解禁mprotect。解决办法只有暂时关闭selinux。

## 使用mmap

《Unix高级编程 第三版》的14.8节中讲解了`mmap`的用法。为了随时回忆，我在这里搬运一下：

```C
#include <sys/mman.h>
void* mmap(void* addr, size_t len, int prot, int flag, int fd, off_t off);
```

* `addr` - 起始地址。通常指定为NULL，这表示由操作系统选择映射区的起始地址。通常被要求是系统虚拟存储页的长度的倍数。
* `len` - 映射区的长度。
* `prot` - 映射区的保护要求。有可读、可写、可执行、不可访问四种权限。依次为`PROT_READ`，`PROT_WRITE`，`PROT_EXEC`，`PROT_NONE`。
* `flag` - 映射方式。有固定、共享、私有等**至少**三种映射方式。依次为`MAP_FIXED`，`MAP_SHARED`，`MAP_PRIVATE`。
* `fd` - 被映射的文件描述符。
* `off` - 被映射的文件的偏移量。通常被要求是系统虚拟存储页的长度的倍数。

不得不提的是，APUE没提到匿名映射（`MAP_ANONYMOUS`）。但APUE中有这样一句话

> 每种实现都可能还有另外一些MAP_xxx标志值，它们是那种实现所特有的。

所谓匿名映射，就是没有指定源文件的映射。实际上匿名映射的底层原理就是把`/dev/zero`当成源文件。

mmap的匿名映射经常被用来代替malloc来更高效地申请超大内存（详见[知乎讨论](https://www.zhihu.com/question/57653599)），并保证申请到的内存是以0填充的。而且使用`mmap`不会被selinux禁掉，比`aligned_alloc + mprotect`的组合更省事。

> malloc有一个chunk仓库管理128字节以下的小块。chunk仓库是为了避免多次系统调用，快速获取内存块，因为从用户态到内核态的转换很耗时（具体包括改变cs、ds、ss，以及各种参数压栈，等等），而有了chunk仓库可以直接在用户态获取内存块。chunk仓库类似链表数组，其块大小由2的n次方构成，最大的块是128字节。
>
> malloc如果分配超过了128字节将调用brk系统调用（改变堆界限,同时创建vma）。而mmap匿名映射也能直接创建vma结构。

这里使用mmap来映射出一段可读、可写、可执行的内存，并把它的首地址当成函数来调用。要注意的是mmap不一定映射到堆内存。

```C
// compile with --std=gnu99 or other higher GNU standards

#include <stdio.h>
#include <stdint.h>
#include <unistd.h>
#include <string.h>
#include <sys/mman.h>   /* mmap() */

static uint8_t code[] = { 
    0xB8,0x2A,0x00,0x00,0x00,   /* mov  eax,0x2a    */
    0xC3,                       /* ret              */
};

int main(void)
{
    void *p = mmap(NULL, sizeof(code), PROT_READ|PROT_WRITE|PROT_EXEC,
            MAP_PRIVATE|MAP_ANONYMOUS, -1, 0); 
    memcpy(p, code, sizeof(code));
    int (*func)(void) = p;
    printf("(dynamic) code returned %d\n", func());
    return 0;
}
```

延伸阅读：mmap的释放：[`munmap`](https://stackoverflow.com/a/6979904).

# # 如何拷贝一个函数的机器码

以上的解决方案只是把机器码存在字节数组里，现在我想直接获取函数的长度并拷贝整个函数的机器码，该怎么做呢？

在试验了https://stackoverflow.com/questions/4156585/how-to-get-the-length-of-a-function-in-bytes中的众多答案之后，发现用dummy function来获取函数的近似长度是最靠谱的。方法很简单，就是紧跟着待测函数声明一个空函数，用空函数的地址减去待测函数的地址，即为待测函数的近似长度。为什么说近似呢，这是考虑到对齐等平台因素。

以下代码就是把一个函数的机器码拷贝到一段mmap出来的区域、并执行它。

```C
#include <stdio.h>
#include <stdint.h>
#include <unistd.h>
#include <string.h>
#include <sys/mman.h>   /* mmap() */

int foo() { return 42; }

void dummy() {}

int main(void)
{
	size_t func_len = (uintptr_t)dummy - (uintptr_t)foo;
	printf("The length of foo() is approx %d bytes.\n", func_len);
	void *p = mmap(NULL, func_len, PROT_READ|PROT_WRITE|PROT_EXEC,
			MAP_PRIVATE|MAP_ANONYMOUS, -1, 0); 
	memcpy(p, &foo, func_len);

	int (*func)(void) = p;
	printf("(dynamic) code returned %d\n", func());

	return 0;
}
```

