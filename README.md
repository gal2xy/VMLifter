# VMLifter

**VMLifter** 是一个基于指令执行轨迹的语义提升工具，专注于值依赖驱动的程序语义重建。该工具利用 ARM / ARM64 的 Load–Store 架构特性，对轨迹进行以内存访问为边界的切片分析。每个切片随后被提升为表达式，并在值依赖分析的基础上进行表达式压缩，最终生成同时包含符号信息与具体数值的表达式。VMLifter 特别适合分析基于虚拟化的混淆（例如 VMP）。对于**VMLifter**生成的表达式结果，可以配合大模型进行分析。

# 原理
详细请见(二选一)：
https://gal2xy.github.io/2026/03/01/%E9%80%86%E5%90%91%E5%AE%9E%E6%88%98/VMLifter/
https://bbs.kanxue.com/thread-290131.htm

# 用法

```shell
python main.py [-h] -v VM_ADDR -f FILE [-d]
```

参数解释如下：

```
-h, --help            				帮助
-v VM_ADDR, --vm-addr VM_ADDR 		虚拟栈的基地址(例如0x127000)
-f FILE, --file FILE  				unidbg trace的指令轨迹文件路径
-d, --debug           				展示切片的详细语义提升过程
```



# 例子

## sub_1cc3c0

基于trace得到的指令轨迹，人为分析的结果如下：

```python
url = "/api/v4/pdp/hot_sales/get?apm_fs=false&apm_p=1.16&apm_pid=shopee%2F%40shopee-rn%2Fproduct-page%2FPRODUCT_PAGE&apm_ts=1734212512053&item_id=27004928820&limit=15&offset=0&shop_id=1205969633"
key1 = 0
const = 0x334b
mod = 0xffffffff + 1
for alp in url:
    key1 = (key1*const) % mod
    key1 = (key1 + ord(alp)) % mod
    # print(hex(key1), hex(ord(alp)))

print(hex(key1))
```

通过VMLifter进行语义提升后，将结果喂给大模型，结果如下：

````
用户提问：
我所上传的文件中的内容为某一函数的VMP指令轨迹语义提升后表达式，请根据这些表达式，识别出参数生成算法，并还原成python代码。对于输入参数，完整展示参数的所有数据，如果参数是可打印的字符串，则将其显示为字符串。

Gemini回答：
根据您提供的 VMP 指令轨迹提升后的表达式，该函数实现了一个基于 **线性同余或类似哈希迭代** 的参数生成算法。

### 1. 算法识别与分析

通过观察表达式的迭代模式 ：

* 
**初始状态**：`mem[0xe4ffdc84]_1` 为 `0x0`，乘数因子 `mem[0xe4ffdc7c]_1` 为 `0x334b` 。


* 
**迭代公式**：`new_value = (multiplier * old_value) + input_byte` 。


* 
**输入来源**：算法按顺序读取位于 `mem[0x12709000]` 开始的内存序列（即参数字节流） 。


* **迭代过程**：
1. 
`0x334b * 0x0 + 0x2f = 0x2f` 


2. 
`0x334b * 0x2f + 0x61 = 0x96b26` 


3. 
`0x334b * 0x96b26 + 0x70 = 0xe31af692` 
...以此类推。





这是一种典型的字符串/序列哈希处理逻辑，其中 `0x334b` (13131) 是常用的哈希乘数。

### 2. 参数数据还原

根据 `mem[0x127090xx]` 地址处的偏移量，提取出的原始输入参数（字节序列）如下 ：

| 地址偏移 | 十六进制值 | ASCII 字符 |
| --- | --- | --- |
| +0x00 | 0x2f | / |
| +0x01 | 0x61 | a |
| +0x02 | 0x70 | p |
| +0x03 | 0x69 | i |
| +0x04 | 0x2f | / |
| +0x05 | 0x76 | v |
| +0x06 | 0x34 | 4 |
| +0x07 | 0x2f | / |
| +0x08 | 0x70 | p |
| +0x09 | 0x64 | d |
| +0x0a | 0x70 | p |
| +0x0b | 0x2f | / |
| +0x0c | 0x68 | h |
| +0x0d | 0x6f | o |
| +0x0e | 0x74 | t |
| +0x0f | 0x5f | _ |
| +0x10 | 0x73 | s |
| +0x11 | 0x61 | a |
| +0x12 | 0x6c | l |
| +0x13 | 0x65 | e |
| +0x14 | 0x73 | s |
| +0x15 | 0x2f | / |
| +0x16 | 0x67 | g |
| +0x17 | 0x65 | e |
| +0x18 | 0x74 | t |
| +0x19 | 0x3f | ? |
| +0x1a | 0x61 | a |
| +0x1b | 0x70 | p |
| +0x1c | 0x6d | m |
| +0x1d | 0x5f | _ |
| +0x1e | 0x66 | f |
| +0x1f | 0x73 | s |
| +0x20 | 0x3d | = |
| +0x21 | 0x66 | f |
| +0x22 | 0x61 | a |
| +0x23 | 0x6c | l |
| +0x24 | 0x73 | s |
| +0x25 | 0x65 | e |
| +0x26 | 0x26 | & |
| +0x27 | 0x61 | a |
| +0x28 | 0x70 | p |
| +0x29 | 0x6d | m |
| +0x2a | 0x5f | _ |
| +0x2b | 0x70 | p |
| +0x2c | 0x3d | = |
| +0x2d | 0x31 | 1 |
| +0x2e | 0x2e | . |
| +0x2f | 0x31 | 1 |

**完整输入参数字符串：**


`/api/v4/pdp/hot_sales/get?apm_fs=false&apm_p=1.1` 

### 3. Python 代码还原

```python
def vmp_parameter_algorithm(input_data):
    """
    还原后的参数生成/哈希算法
    :param input_data: 输入的字节列表或字符串
    :return: 最终生成的计算值 (32位无符号整数)
    """
    if isinstance(input_data, str):
        input_bytes = input_data.encode('ascii')
    else:
        input_bytes = input_data

    multiplier = 0x334b  # 13131
    current_value = 0x0
    
    for byte in input_bytes:
        # 模拟表达式中的迭代过程: (multiplier * old_value) + current_byte
        # 并保持在 32 位无符号整数范围内
        current_value = (multiplier * current_value + byte) & 0xFFFFFFFF
        
    return current_value

# 输入数据还原
raw_params = "/api/v4/pdp/hot_sales/get?apm_fs=false&apm_p=1.1"
result = vmp_parameter_algorithm(raw_params)

print(f"输入参数字符串: {raw_params}")
print(f"最终生成结果: {hex(result)}")

```
````






