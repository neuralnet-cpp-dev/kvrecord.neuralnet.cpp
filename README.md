# kvrecord.neuralnet.cpp

neuralnet.cpp 生态的**自描述二进制键值记录**容器（header-only，轻量 JSON 替代）。

| 头文件 | 内容 |
|---|---|
| `model_keyvalue_record.hpp` | `KeyValueRecord`：UInt / Str / UIntArray / 嵌套 Record；v1 内联 + v2 地址布局；版本自举解析；未知类型按长度跳过（前向兼容） |

## 用法

```cpp
#include <neuralnet.cpp/model_keyvalue_record.hpp>

nn::KeyValueRecord rec;
rec.set("answer", std::uint64_t{42});
std::uint64_t v = 0;
(void)rec.get("answer", v);
auto bytes = rec.serialize_v2();
auto parsed = nn::KeyValueRecord::parse(bytes);
```

## 依赖

依赖 [`base.neuralnet.cpp`](https://github.com/neuralnet-cpp-dev/base.neuralnet.cpp)（`Result<T>`），以 submodule 位于 `extern/base`。

```bash
git clone --recurse-submodules .../kvrecord.neuralnet.cpp
```

CMake：

```cmake
add_subdirectory(kvrecord)   # 自动拉起 extern/base
target_link_libraries(app PRIVATE neuralnet::kvrecord)
```
