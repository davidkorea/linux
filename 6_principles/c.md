


# 1. 结构介绍
```c
//<my_header_file.h>

#define enable 1
```
```c
#include <stdio.h>
#include <stdlib.h>
#include <my_header_file.h>

#define PI 3.1415926

#if enable
float cube(float x);          // 由于c从上往下编译，需要将下面定义的函数提前放在上面声明一下

int main(void)
{
  float x;
  float V;
  printf("please input a number(float) \n");
  scanf("%f",&x);           // 通过 &，将键盘输入赋值给变量
  V = cube(x);
  printf("The V is %.2f",V);
  system("pause");
  return 0
}

float cube(float x)
{
  float V;
  V = x * x * x;
  return V
}
#endif
```
- `#include`引用头文件
- `#define`定义宏，下面函数中可以直接调用
  - <my_header_file.h>自定义的头文件中，包含一个宏定义
  - c文件中使用如下结构,来根据头文件中宏的数值0或1来判断其中代码是否有效
    ```c
    #if enable
    funtions(){}
    #endif
    ```
- 函数定义 `返回值类型 函数名(类型1 变量名1，... 类型n 变量n){}`
  - `int main(void)`中的int：返回值类型，即return的0为整型
  - 如果定一个没有返回值的函数，则可以使用void来定义，即`void my_func(int a, int b)`
- `main`函数不能被其他函数调用，也不能被自己调用


# 2. 条件判断

> 场景
> - 用水量<50, 价格 = 0.5 x 用水量
> - 50<=用水量<100, 价格 = 0.6 x 用水量 + 10
> - 用水量>=100, 价格 = 0.7 x 用水量 + 20

### if...else
```c

int main(void)
{
  float consume;
  float price;
  
  printf("please input water consuption: ")
  scanf("%f",&consume)
    
  if (consume<50)
  {
    price = 0.5 * consume;
  }
  else if (consume<100)
  {
    price = 0.6 * consume + 10;
  }
  else
  {
    price = 0.7 * consume +20
  }
  printf("the price is %.2f",price);
  system("pause");
  return 0
}
```

### switch...case
```c
switch (表达式)        // 此处表达式不使用浮点型，使用整型，字符型或枚举型
{
  case 常量表达式1:
    语句1；
   break；
   case 常量表达式2:
    语句2；
   break；
   default 常量表达式:
    语句；
   break；
}
```
```c
int main(void)
{
  float consume;
  float price;
  int level;
  
  printf("please input water consuption: ");
  scanf("%f",&consume);
  
  level = consume/50;   //取整，小于50=0，50～100=1
  
  switch (level)
  {
    case 0:                           // if level=0
      price = 0.5 * consume;
      break;    
    case 1:
      price = 0.6 * consume + 10;
      break;
    default:                          // if level!=0, !=1, others
      price = 0.7 * consume + 20;
      break;
  }
  system("pause");
}
```

# 3. 变量
### 3.1. 全局变量 vs 局部变量 vs 外部变量 vs 自动变量
- 全局变量，定义在寒暑体外面，所有函数都可以调用
- 局部变量，定义在某一个函数的内部，仅该函数自己可以使用
- 外部变量，不同文件之间的调用
- 自动变量，自动识别变量类型
```c
extern x=100;       // 外部变量，其他文件中的函数可以调用该变量
int a=5;            // 全局变量

void main()
{
  int b=10;         // 局部变量
  auto c= 0.5;      // c is float, 局部变量
  auto d= 3;        // d is int, 局部变量
}
```

### 3.2 静态变量
- 静态变量，整个程序执行期间，静态变量的最后一次赋值，会被保存在内存中。而非静态变量，每次调用时，都会重新开辟一块内存空间，并使用初始的默认赋值

```c
void test()
{
  static int a = 10；    // 静态变量
  auto b = 1;
  a++;
  b++;
  printf("a = %d \n",a);
  printf("b = %d \n",b);
}

void main()
for (i=0, i<3, i++)
{
  test();
}
```
```
a = 10
b = 2

a = 11
b = 2

a = 12
b = 2
```

### 库函数
标准库，三方库，自己库































