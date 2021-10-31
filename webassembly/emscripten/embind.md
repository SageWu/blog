# embind

连接C/C++和JavaScript。唯一限制是不支持拥有复杂生命周期语义的野指针。\
编译时，使用`--bind`参数。

## 函数

```cpp
#include <emscripten/bind.h>

using namespace emscripten;

float add(float a, float b) {
  return a + b;
}

EMSCRIPTEN_BINDINGS(my_module) {
  function("add", &add);
}
```

### 重载

```cpp
#include <emscripten/bind.h>

using namespace emscripten;

void foo();
void foo(int i);
void foo(float f)；

EMSCRIPTEN_BINDINGS(my_module) {
  function("foo", select_overload<void()>(&foo));
  function("foo_int", select_overload<void(int)>(&foo));
  function("foo_float", select_overload<void(float)>(&foo));
}
```

## 类

```cpp
#include <string>
#include <emscripten/bind.h>

using namespace emscritpen;

class MyClass {
private:
  int _x;
  std::string _str;

public:
  static std::string getStringFromInstance(const MyClass& instance) {
    return instance._str;
  }

  MyClass(int x, std::string str): _x(x), _str(str) {

  }
  void incrementX() {
    ++_x;
  }
  int getX() const {
    return _x;
  }
  void setX(int x) {
    _x = x;
  }
};

EMSCRIPTEN_BINDINGS(my_class_example) {
  class_<MyClass>("MyClass")
    .class_function("getStringFromInstance", &MyClass::getStringFromInstance)
    .constructor<int, std::string>()
    .property("_x", &MyClass::getX, &MyClass::setX)
    .function("incrementX", &MyClass::incrementX)
    ;
}
```

> 注意：在JS中创建的C++对象或返回给JS的，需手动销毁，不然将导致堆不断增长。

### 克隆

克隆handle，会使得底层C++对象的引用计数增加。只有当引用计数为0时，C++对象才真的被销毁。

```js
const anotherHandle = handle.clone();
handle.delete();
anotherHandle.delete();
```

## 值类型

手动管理基础类型的内存比较繁琐，emscripten提供了`value_array`和`value_object`。

```c++
struct Point2f {
  float x;
  float y;
};
struct PersonRecord {
  std::string name;
  int age;
};
PersonRecord findPersonAtLocation(Point2f);

EMSCRIPTEN_BINDINGS(my_value_example) {
  value_array<Point2f>("Point2f")
    .element(&Point2f::x)
    .element(&Point2f::y);

  value_object<PersonRecord>("PersonRecord")
    .field("name", &PersonRecord::name)
    .field("age", &PersonRecord::age);

  function("findPersonAtLocation", &findPersonAtLocation);
}
```

```js
let person = Module.findPersonAtLocation([1, 2]);
```

## 枚举

```cpp
enum OldStyle {
  OLD_STYLE_ONE,
  OLD_STYLE_TWO
};

enum class NewStyle {
  ONE,
  TWO
};

EMSCRIPTEN_BINDINGS(my_enum_example) {
  enum_<OldStyle>("OldStyle")
    .value("ONE", OLD_STYLE_ONE)
    .value("TWO", OLD_STYLE_TWO);
  enum_<NewStyle>("NewStyle")
    .value("ONE", NewStyle::ONE)
    .value("TWO", NewStyle::TWO);
}
```

## 常量

```cpp
EMSCRIPTEN_BINDINGS(my_constant_example) {
  constant("SOME_CONSTANT", SOME_CONSTANT);
}
```
