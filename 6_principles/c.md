


1. 结构介绍
```c
#include <stdio.h>
#include <stdlib.h>
#include <my_header_file.h>

#define PI 3.1415926

int main(void)
{
  float a=1;
  printf("the number is %.2f",a)
  system("pause")
  return 0
}
```
- `#include`引用头文件
- `#define`定义宏，下面函数中可以直接调用
- 函数定义 `返回值类型 函数名(类型1 变量名1，... 类型n 变量n){}`
  - `int main(void)`中的int：返回值类型，即return的0为整型
  - 如果定一个没有返回值的函数，则可以使用void来定义，即`void my_func(int a, int b)`
