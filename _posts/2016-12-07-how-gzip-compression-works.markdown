---
title: How GZIP compression works
published: true
layout: post
---

GZIP provides lossless compression, which in essence means we can recover the original data when decompressing it. It is based on the DEFLATE algorithm, a combination of LZ77 algorithm and Huffman encoding.

### LZ77 Algorithm

The LZ77 algorithm simply replaces repeated occurences of the data with references to the original. This method of compression has varying results for different documents, with highly repetitive documents benefitting greatly from this approach. A matchi is encoded by a pair of numbers called "length-distance pair" which is best described as:

> "each of the next length of characters is equal to the characters exactly distance characters behind it in the uncompressed stream"

### Huffman Encoding

This is a variable-length encoding method which assigns the shortest frequency codes to the most frequently occuring characters within a string.

Below is a comparison of a simple phrase in regular ASCII encoding and also Huffman coding:

```
Original Text: The quick brown fox jumps over the lazy dog

ASCII Encoded: 01010100 01101000 01100101 00100000 01110001 01110101 01101001 01100011 01101011 00100000 01100010 01110010 01101111 01110111 01101110 00100000 01100110 01101111 01111000 00100000 01101010 01110101 01101101 01110000 01110011 00100000 01101111 01110110 01100101 01110010 00100000 01110100 01101000 01100101 00100000 01101100 01100001 01111010 01111001 00100000 01100100 01101111 01100111

Huffman Encoded: 01110 10110 1010 110 01100 0100 00001 01010 111101 110 111110 1001 001 111111 00000 110 00011 001 111100 110 111010 0100 101111 01111 111000 110 001 111011 1010 1001 110 01011 10110 1010 110 10001 111001 00010 101110 110 10000 001 01101 
```
You can see from the above how much shorter the Huffman encoding is in terms of absolute binary. Below is a comparison of how the encoding method performs.

#### Memory requirements:
* ASCII: 344 bit
* Huffman: 237 bit

#### Average code length:
* ASCII: 8 bit 
* Huffman: 4.5116279069767 bit

**Final Compression rate: 0.56395348837209**

### So is GZIP the best compression algorithm?

**No.**

In future posts, I aim to explore and compare how best to compress huge TB size weblogs which are written once and used many times, from applications such as real time bidding systems.
