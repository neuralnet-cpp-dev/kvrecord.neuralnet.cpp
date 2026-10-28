# kvrecord.neuralnet.cpp

neuralnet.cpp 生态的**自描述二进制键值记录**容器（header-only，轻量 JSON 替代）。

| 头文件 | 内容 |
|---|---|
| `model_keyvalue_record.hpp` | `KeyValueRecord`：UInt / Str / UIntArray / 嵌套 Record；v1 内联 + v2 地址布局；版本自举解析；未知类型按长度跳过（前向兼容） |

## 格式

- **v1**（`serialize()`）：内联布局，无版本哨兵。模型 spec 头等既有格式继续用；
  `Record` 字段 v1 不支持（跳过不写）。
- **v2**（`serialize_v2(base_offset=0)`）：自包含字节块 = 字段表 + 连续数据块，
  字段地址 = `base_offset` + 块内相对位移；嵌套 Record 子块递归序列化，
  addr 恒为**文件绝对偏移**（大文件可经 `parse_at(ReadAt, offset)` 流式解析）。
- **版本自举**：`parse` 看哨兵字段 `kvrec_version`——无 = v1，有 = 按版本分派。
  未知字段类型按长度跳过（前向兼容）。
- ⚠ `kvrec_version` 是 v2 保留键名，业务字段不得使用。

## API 速览

```cpp
#include <neuralnet.cpp/model_keyvalue_record.hpp>

nn::KeyValueRecord rec;
rec.set("answer", std::uint64_t{42});                    // UInt
rec.set("name", std::string{"mnist"});                   // Str
rec.set("shape", std::vector<std::uint64_t>{1, 28, 28}); // UIntArray
nn::KeyValueRecord nested;
nested.set("sha256", std::string{"..."});
rec.set("source", nested);                               // 嵌套 Record（v2）

// 读取：返回 false = 缺失或类型不符
std::uint64_t v = 0;
bool ok = rec.get("answer", v);                          // ok && v == 42
bool has = rec.has("name");

// 序列化 / 解析
std::string bytes_v1 = rec.serialize();                  // v1（嵌套 Record 跳过）
std::string bytes_v2 = rec.serialize_v2();               // v2 自包含
auto parsed = nn::KeyValueRecord::parse(bytes_v2);       // Result<KeyValueRecord>
```

## 用法 / CMake

```cmake
add_subdirectory(kvrecord)   # 自动拉起 extern/base（若父工程未提供）
target_link_libraries(app PRIVATE neuralnet::kvrecord)   # 提供 include + C++23
```

## 依赖

依赖 [`base.neuralnet.cpp`](https://github.com/neuralnet-cpp-dev/base.neuralnet.cpp)（`Result<T>`），以 submodule 位于 `extern/base`。

```bash
git clone --recurse-submodules .../kvrecord.neuralnet.cpp
# 或已 clone 后：git submodule update --init --recursive
```

## 构建与测试

```bash
git submodule update --init --recursive
cmake -B build
cmake --build build
ctest --test-dir build --output-on-failure
```

| 测试 | 覆盖 |
|---|---|
| `kvrecord_kvrec_test` | v1 round-trip（含空记录）/ 版本自举（无哨兵=v1、有=v2，同一 `parse` 入口分派）/ v2 round-trip（UInt/Str/UIntArray + 多层嵌套）/ v2 未知类型按长度跳过（前向兼容）/ 整体搬迁的 v2 地址失效 → 硬报错（不静默错值） |

被父工程 `add_subdirectory` 消费时测试**默认不构建**（`NN_BUILD_TESTS` 默认值 =
是否顶层工程）；显式 `-DNN_BUILD_TESTS=ON` 可在嵌入时强制打开。

许可证：LGPL-3.0（见 [LICENSE](LICENSE)）。

## 格式规范与 Python 参考实现

- **字节级规范**：[docs/kvrec-format-spec.md](docs/kvrec-format-spec.md)（v1/v2 布局、
  addr 连续性、版本自举）。v2 起格式**稳定**，演进只经新增版本哨兵值。
- **Python 参考实现**：[python/kvrecord.py](python/kvrecord.py)（单文件、仅标准库）——
  纯 Python 数据管线可原生读写 `.nnvocab` / `.nndataset` 等容器，无需 subprocess 桥接 C++ CLI。
  合规口径：`dumps_v2()` 与 C++ `serialize_v2()` **逐字节一致**（对 `bpe_vocab_stories.nnvocab`
  等真实产物往返验证通过）。

```python
import sys; sys.path.insert(0, "python")
from kvrecord import KVRecord
r = KVRecord.load("bpe_vocab_stories.nnvocab")   # 嗅探 NNVC 容器 / 裸 kvrec
vocab = r.get_bytes("vocab")                     # 原始字节（非 UTF-8 安全）
r.set_u64("added", 1).save("out.nnvocab")        # 重写仍是逐字节兼容的 v2
```
