# 数据压缩
## 源代码下载：[codes.zip](https://github.com/user-attachments/files/16844970/codes.zip)
##   email：[zehaiyu589@gmail.com](mailto:zehaiyu589@gmail.com)


## 1.Huffman coding:
 在哈夫曼编码中，首先会统计文件中各字符出现的次数来代表权重，而权重大的字符出现次数更多，用以更短的编码序列（也就是更靠近根节点的层次）可以让压缩后的数据规模更小

首先声明数据类型并创建树
```python
# 构建哈夫曼树
from collections import Counter
import heapq


class Node:
    def __init__(self, char, freq):
        self.char = char
        self.freq = freq
        self.left = None
        self.right = None

    # 定义比较函数，使得Node类可以在优先队列中排序
    def __lt__(self, other):
        return self.freq < other.freq


def build_huffman_tree(text_build):
    frequency = Counter(text_build)
    heap = [Node(char, freq) for char, freq in frequency.items()]
    heapq.heapify(heap)

    while len(heap) > 1:
        node1 = heapq.heappop(heap)
        node2 = heapq.heappop(heap)
        merged = Node(None, node1.freq + node2.freq)
        merged.left = node1
        merged.right = node2
        heapq.heappush(heap, merged)

    return heap[0]

```
哈夫曼编码表即是根据数据中不同元素出现频率来生成树之后，树中每一个字符代表的编码
```python
# 创建哈夫曼编码表
def build_codes_table(root_build):
    codes = {}

    def build_codes(node, current_code):
        if node is None:
            return

        if node.char is not None:
            codes[node.char] = current_code

        build_codes(node.left, current_code + "0")
        build_codes(node.right, current_code + "1")

    build_codes(root_build, "")
    return codes
```
形如：
![image](https://github.com/user-attachments/assets/00824b59-fc1a-4177-b396-aa49990dd709)

此处就是简单的将table中的字符与编码互换，得到二进制的压缩文件，也就是我们常说的zip或者rar文件
```python
# 压缩数据
def huffman_compress(text_compress):
    root_compress = build_huffman_tree(text_compress)
    codes_table_c = build_codes_table(root_compress)
    encoded_text = ''.join(codes_table_c[char] for char in text_compress)
    return encoded_text, codes_table_c, root_compress


# 解压数据
def huffman_decompress(encoded_text, root_decompress):
    decoded_text = []
    node = root_decompress
    for bit in encoded_text:
        if bit == '0':
            node = node.left
        else:
            node = node.right

        if node.left is None and node.right is None:
            decoded_text.append(node.char)
            node = root_decompress

    return ''.join(decoded_text)
```
压缩后数据文件如下：
![image](https://github.com/user-attachments/assets/9ba4d5a1-d611-431a-b5b3-f7d744136ca9)

可值得一提的是现在使用的压缩工具中，ZIP格式通常使用DEFLATE算法，这是一种结合了LZ77算法和哈夫曼编码的压缩方法。LZ77算法用于识别数据中的重复模式，而哈夫曼编码则用于将这些模式转换为更短的位序列，从而实现压缩。
rRAR格式使用的是其专有的压缩算法，通常是基于LZ压缩算法的变体，如LZ77或LZMA（Lempel-Ziv-Markov chain Algorithm）。LZMA算法是一种更先进的压缩技术，它使用基于字典的压缩方法，而并非哈夫曼编码。
 




## 2.DEFLATE 与 LZMA

### 那么接下来我们可以观察lz77算法与常规哈夫曼的不同之处
![image](https://github.com/user-attachments/assets/f3c25ba1-34e1-48c6-b0d5-36859e3644e4)
这里Python 标准库中提供了 zlib 模块，它实现了 DEFLATE 算法。我们可以调用该库并运行：
```python
import zlib


# 从文件中读取文本数据，使用 utf-8 编码
def read_text_from_file(file_path):
    with open(file_path, 'r', encoding='utf-8') as file:
        return file.read()


# 将字节数据写入文件（以二进制模式）
def write_bytes_to_file(data, file_path):
    with open(file_path, 'wb') as file:
        file.write(data)


# 待压缩的数据（以字符串为例）
idata = read_text_from_file('source.txt')


# 将数据转换为字节格式
data_bytes = idata.encode('utf-8')

# 压缩数据，压缩级别范围从 0（不压缩）到 9（最大压缩）
compressed_data = zlib.compress(data_bytes, level=6)

# 输出压缩后的数据
print("压缩后的数据：", compressed_data)
write_bytes_to_file(compressed_data, 'deflate/deflate_com.bin')

# 解压缩数据
decompressed_data = zlib.decompress(compressed_data)

write_bytes_to_file(decompressed_data, 'deflate/deflate_decom.txt')


# 检查解压缩后的数据是否与原始数据相同
assert idata == decompressed_data.decode('utf-8'), "解压缩后的数据与原始数据不匹配！"
```
代码较为简单，需要注意的点时zlib.compress参数只支持字节类型而不支持str（data）类型，这同样意味着压缩后数据不能够以str形式存储到txt等文件中，需要存入例如bin的二进制文件并查看，也可以直接进行print，会打印出代表16进制数的转义字符：
![image](https://github.com/user-attachments/assets/49e36652-02e7-4367-818b-b2f5288d43be)

使用Hex Editor Neo查看文件内容也是同理：
![image](https://github.com/user-attachments/assets/3c37d0dc-302b-4699-9b51-844d2183845d)


最后是rar的lzma算法，也可以在调库中使用，生成数据压缩文件为xz类型，在此不进行赘述，一并附加在源代码下载链接中。