# Promise

```cpp
std::promise<int> p;
std::future<int> f = p.get_future();

// other thread
p.set_value(0);

// main thread
f.get();
```
