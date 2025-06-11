+++
title = 'Makefile简单的示例'
date = 2025-06-11T20:13:33+08:00
draft = true
+++


> 使用的是Ubuntu系统
---

<!--more-->


* 代码程序 `main.c`

```c
#include <stdio.h>

int main(void)
{

    printf("Hello world\n");

    return 0;
}
```

* `makefile`文件
```makefile
# 定义编译器
CC = gcc

# 编译选项
CFLAGS = -Wall -g

# 最终目标可执行文件
TARGET = app

# 所有源文件
SRCS = main.c 

# 生成对应的目标文件列表
OBJS = $(SRCS:.c=.o)

# 默认目标：构建可执行文件
all: $(TARGET)

# 链接目标文件生成可执行文件
$(TARGET): $(OBJS)
	$(CC) $(CFLAGS) -o $@ $^

# 编译规则：从.c生成.o
%.o: %.c
	$(CC) $(CFLAGS) -c $< -o $@

run: $(TARGET)
	./$(TARGET)

# 清理生成的文件
clean:
	rm -f $(OBJS) $(TARGET)

# 声明伪目标（不生成实际文件）
.PHONY: all clean
```

* 执行 `make all`生成的文件路径

```markdown

ubuntu% tree
.
├── app
├── main.c
├── main.o
└── makefile
```
