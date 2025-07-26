# Week 2


## Step 1: Tokenization

We first create a full tokenizer class which would implement two methods: tokenization of the text (encoding) and an inverse operation to bring back the text (decoding).

1. The training data is split into a list of tokens (not yet numbered). These may include words, punctuation marks, whitespaces (this is optional) etc.
2. A __vocabulary__ is built which arranges __unique__ words from the training set in alphabetical order and indexes them in that order. The indices are called __token IDs__.


### Special Context Tokens

Two extra tokens, __<|unk|>__ and __<|endoftext|>__ are added to the end of the vocabulary. These are __special context__ tokens; <|unk|> (unknown) is for a word not found in the vocabulary and <|endoftext|> is used to separate different sources/blocks of text (we may be feeding multiple sources to the training model together).

There also exist other types of special context tokens, such as __[BOS]__ (beginning of sequence), __[EOS]__ (end of sequence) and __[PAD]__ (padding to make batch sizes of texts equal). GPT models only use <|endoftext|>.


### Tokenization Algorithms

1. __Word Based__: This tokenization scheme converts entire words into tokens. Drawbacks:
    - OOV (Out Of Vocabulary) tokens can't be dealt with properly (have to be replaced by <|unk|>).
    - A large-sized vocabulary (~200,000 entries) would be developed if the entire English language were covered.
    - Would fail to capture relation between close words (eg. 'boy' and 'boys'.)

2. __Character Based__: Converts each character into a token.
    - Extremely few OOV tokens would occur.
    - The vocabulary size would also be small, leading to efficient computation.
    - However, the meaning associated with the word would be completely lost.
    - Also, the tokenized sequence is much longer than the initial raw text.

A nice balance between these schemes would be a __subword-based__ algorithm:

3. __Subword Based__: It follows two rules.
    - Frequently used words are not split.
    - Rare words are split into more meaningful subwords.

### Byte Pair Encoding

The algorithm is as follows: 

```
repeat until no byte pair occurs more than once {
   replace the byte pair occuring most frequently with a byte not in the dataset
}
```
Here is an example:
```
aaabdaaabac aa->Z
ZabdZabac   ab->Y
ZYdZYac     ZY->X
XdXac       --
```

This algorithm is useful in LLMs because it will ensure that all frequently used words are assigned a single token (and character-based encoding may subsequently follow; making tokenization easier since subword tokenization may be harder to implement).

### Implementing BPE in LLMs

Consider the example `["old":9 , "older":3 , "finest":9 , "lowest":4]` (number infront of the colon indicates the number of occurences of that word in the dataset). This is the dataset to be tokenized.
1. Preprocessing: We need to add an end token `</w>` at the end of each word: `["old</w>":9 , "older</w>":3 , "finest</w>":9 , "lowest</w>":4]`
2. We will now create a frequency table for each character.

| Character | Frequency |
| --------- | --------- |
| `</w>`    | 23        |
| o         | 14        |
| l         | 14        |
| d         | 10        |
| e         | 16        |
| r         | 3         |
| f         | 9         |
| i         | 9         |
| n         | 9         |
| s         | 13        |
| t         | 13        |
| w         | 4         |

