# Compound Types

## Array

```cpp
// 数组元素个数需为常数、常量或常数表达式
int array[4];

const int n = 4;
int array[n];

int array[n + 2];
```

```cpp
// 只初始化前部分，后部分都初始化为 0
int array[4] = { 0 };
```

## String

`\0`作为字符串结尾。

```cpp
char str[4] = { 'a', 'b', 'c' };

char str[] = "abc";
```

```cpp
// 空格隔开的字符串常量会进行连接
std::cout << "hello"
" world";
```

```cpp
// cin 默认读取单个词汇
// getline(array, limit) 读取整行，去除换行字符
cin.getline(array, 10);
// get(array, limit) 读取整行，保留换行字符
cin.get(array, 10);
```

`string`类

* 自动维护容器大小，更安全
* 可直接赋值（C对应`strcpy`, `strncpy`）
* 可直接拼接（C对应`strcat`, `strncat`）

```cpp
std::cout << R"(hello world)";
std::cout << R"+*(hello world)+*";
```

## Structure

```cpp
struct Player {
  int a;  // member
};

struct Player p1; // c style
Player p1;

// memberwise assignment
Player p2 = p1;
```

位成员，指定成员使用的位数

```cpp
struct Register {
  unsigned int a: 4;
  unsigned int : 4;
  bool c: 1;
  bool d: 1;
}

Register reg = { 4, true, false };
```
