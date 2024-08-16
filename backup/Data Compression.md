# 数据压缩

## Huffman coding:
 在哈夫曼编码中，首先会统计文件中各字符出现的次数来代表权重，而权重大的字符出现次数更多，用以更短的编码序列（也就是更靠近根节点的层次）可以让压缩后的数据规模更小


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


# 从文件中读取文本数据，使用 utf-8 编码
def read_text_from_file(file_path):
    with open(file_path, 'r', encoding='utf-8') as file:
        return file.read()


# 将数据写入文件，使用 utf-8 编码
def write_to_file(data, file_path):
    with open(file_path, 'w', encoding='utf-8') as file:
        file.write(data)


# 示例使用
if __name__ == "__main__":
    # 读取输入文件
    input_text = read_text_from_file('source.txt')

    # 压缩数据
    compressed, codes_table, root = huffman_compress(input_text)

    # 写入压缩后的数据和编码表到文件
    write_to_file(compressed, 'compressed.txt')
    write_to_file(str(codes_table), 'codes_table.txt')

    # 从文件中读取压缩数据进行解压缩（这里为了演示，我们直接使用变量）
    compressed_data = read_text_from_file('compressed.txt')

    # 解压数据
    decompressed = huffman_decompress(compressed_data, root)

    # 写入解压缩后的数据到文件
    write_to_file(decompressed, 'decompressed.txt')

    print("Compression and decompression completed.")

`