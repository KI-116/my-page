# tokenizer

## conception

unicode->tokenizer->token ids(int list)->model->token ids->tokenizer->unicode

utf-8是最常见的unicode编码方式，能够表示世界上几乎所有的字符。

indices: 得到的token ids，以列表的形式表示

``` python
indices = tokenizer.encode("Hello, how are you?")
reconstructed_text = tokenizer.decode(indices)
assert reconstructed_text == "Hello, how are you?"
compression_ratio = len(indices) / len("Hello, how are you?")
```

1 byte 可以表示 256 个字符，2 bytes 可以表示 65536 个字符，3 bytes 可以表示 16777216 个字符，4 bytes 可以表示 4294967296 个字符。

compression ratio: 压缩率，表示token ids的长度与原始文本长度的比值。等于1的时候，attention 是quadratic的。

## BPE： byte pair encoding

如果遇到冷门输入，或过大输入，word-base会unk这样的输入，可能引起OOV问题。character-base会把每个字符都当成一个token，导致输入过长，增加计算成本。


Basic idea: 训练过程中，统计文本中所有相邻字符对的频率，找到出现频率最高的字符对，将其合并成一个新的token。重复这个过程，直到达到预定的词汇表大小。

常见输入：single token
罕见输入：many tokens 



GPT2 paper使用了word base tokenization，把raw text分解成initial segments,然后在每个segment上使用BPE进行tokenization。

也即是，从每个token分配一字节开始，逐渐合并频率最高的token对，直到达到预定的词汇表大小。

``` python
def train_bpe(string: str, num_merges: int) ->BPETokenizerParams:
    indices = list(map(int,string.encode('utf-8'))) # 将字符串编码为UTF-8字节序列，并转换为整数列表
    merges:dict[tuple[int, int], int] = {} # index1, index2 => merged index, 也就是合并后的token id
    vocab: dict[int, bytes] ={x: bytes([x]) for x in range(256)} # index => byte，初始词汇表包含所有单字节字符

    for i in range(num_merges):
        # 统计所有相邻字符对的频率
        counts = defaultdict(int)
        # 对于zip(indices, indices[1:])，可以得到相邻的字符对，例如对于indices=[1, 2, 3]，zip(indices, indices[1:])会生成(1, 2)和(2, 3)两个对。
        for index1, index2 in zip(indices, indices[1:]):
            counts[(index1, index2)] += 1
        
        # 找到出现频率最高的字符对
        pair = max(counts, key=counts.get)
        index1, index2 = pair

        # merge the pair into a new token
        new_index = 256 + i # 新token的id，从256开始
        vocab[new_index] = vocab[index1] + vocab[index2] # 合并后的token对应的字节序列
        indices = merge(indices, pair, new_index) # 将所有出现该pair的地方替换为new_index
    
    return BPETokenizerParams(vocab=vocab, merges=merges)
```

不足：
- encode() currently loops over all merges. Only loop over merges that matter.  
- Detect and preserve special tokens (e.g., <|endoftext|>).  
- Use pre-tokenization (e.g., the GPT-2 tokenizer regex).  
- Try to make the implementation as fast as possible.

解释：
- encode() currently loops over all merges: 在编码过程中，当前实现会遍历所有的合并规则，这可能效率不高。优化方法是只遍历那些实际影响输入文本的合并规则。
- Detect and preserve special tokens: 在训练过程中，需要识别并保留特殊的token（例如 `<|endoftext|>`），以确保它们在编码和解码过程中被正确处理。
- Use pre-tokenization: 在应用BPE之前，可以使用预分词技术（例如GPT-2的正则表达式分词器）来将文本分解成更小的单元，这样可以提高BPE的效率和效果。
- Try to make the implementation as fast as possible: 需要优化实现，使其尽可能高效，减少计算时间和资源消耗。