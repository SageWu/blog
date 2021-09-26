# embind

连接C/C++和JavaScript。绑定C/C++的函数、类、值类型、枚举、常量、指针，唯一限制是不支持拥有复杂生命周期语义的野指针。\
通过`embind`暴漏的符号，可在`Module`上获取。\
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

注意：在JS中创建的C++对象，需手动销毁，不然将导致堆不断增长。
