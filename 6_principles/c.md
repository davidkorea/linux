


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




































