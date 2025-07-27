# Week 2

All content is taken from Vizuara AI's YouTube playlist '[Building LLMs from Scratch](https://youtube.com/playlist?list=PLPTV0NXA_ZSgsLAr8YCgCwhPIJNNtexWu&si=HPSZqC8bKeJmPCje)'.

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

### Modified BPE Algorithm in LLMs

__Note__: The algorithm focuses more on introducing more tokens starting off from character based tokenization rather than splitting words into subwords.

Consider the example `["old":9 , "older":3 , "finest":9 , "lowest":4]` (number infront of the colon indicates the number of occurences of that word in the dataset). This is the dataset to be tokenized.
1. Preprocessing: We need to add an end token `</w>` at the end of each word: `["old</w>":9 , "older</w>":3 , "finest</w>":9 , "lowest</w>":4]`
2. We will now create a frequency table for each token; we shall assume character based encoding.

| Token       | Frequency | Token       | Frequency | Token       | Frequency |
|-----------  |-----------|-----------  |-----------|-----------  |-----------|
| `</w>`      | 23        | `f`         | 9         | `s`         | 13        |
| `o`         | 14        | `i`         | 9         | `t`         | 13        |
| `l`         | 14        | `n`         | 9         | `w`         | 4         |
| `d`         | 10        | `r`         | 3         | `e`         | 16        |

3. We will now use two frequently occuring tokens (here `e` and `s`) and create a new token `es` (prioritizing this over the individual tokens `e` and `s`). Note how `s` has 0 occurences now (_it is still a token!_):

| Token       | Frequency | Token       | Frequency | Token       | Frequency |
|-----------  |-----------|-----------  |-----------|-----------  |-----------|
| `</w>`      | 23        | `f`         | 9         | `es`        | 13        |
| `o`         | 14        | `i`         | 9         | `t`         | 13        |
| `l`         | 14        | `n`         | 9         | `w`         | 4         |
| `d`         | 10        | `r`         | 3         | `e`         | 3         |
| `s`         | 0         |             |           |             |           |


4. We repeat this process until we reach a predefined limit such as:
    - The token count.
    - The number of iterations.
    
The final tokens for our example are:

| Token       | Frequency | Token       | Frequency | Token       | Frequency |
|-----------  |-----------|-----------  |-----------|-----------  |-----------|
| `</w>`      | 10        | `r`         | 3         | `w`         | 4         |
| `o`         | 4         | `f`         | 9         | `est</w>`   | 13        |
| `l`         | 4         | `i`         | 9         | `old`       | 10        |
| `e`         | 3         | `n`         | 9         |             |           |

We did not create a token for `fin` because it only occured in one word.

__Advantage__: This method handles unknown words without the <|unk|> token effortlessly; all 26 English letters are still present as tokens.

### Input-Target Pairs

For a given input to 