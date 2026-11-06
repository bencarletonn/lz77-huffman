# LZ77 + Huffman Compression

A lossless text compressor written in pure Python. It combines **LZ77** dictionary compression with **canonical Huffman coding**, and is tested against a large LaTeX file (`bigbible.tex`, ~1 MB).

`File Report.pdf` gives a brief overview of the techniques used.

## How it works

1. **LZ77** scans the input with a sliding window (search buffer of 2^16 − 2 characters, lookahead of 2^8 − 2) and emits `(offset, length, next_char)` triples.
2. The triples are split into three streams:
   - offsets, stored as 2 bytes each
   - lengths, stored as 1 byte each
   - literal characters, concatenated into a string
3. The literal characters are compressed with **canonical Huffman coding**. The header stores the symbols and their code lengths, so the decoder can rebuild the codes without a tree.
4. As a fallback, the encoder also Huffman-codes the raw input. If that is smaller than the LZ77 output, it is used instead. A leading flag byte tells the decoder which mode was used.

### Output format

| Mode | Layout |
| --- | --- |
| LZ77 + Huffman | flag `0xFF` · 3-byte offset-stream size · offsets · 3-byte length-stream size · lengths · Huffman-coded characters |
| Huffman only | flag `0x80` · Huffman-coded input |

## Files

| File | Purpose |
| --- | --- |
| `encoder.py` | Compresses a `.tex` file to a `.lz` file |
| `decoder.py` | Decompresses a `.lz` file to `<name>-decoded.tex` |
| `testEncoderDecoder.py` | Runs both, checks the round trip, and prints the compression ratio |
| `bigbible.tex` | Sample input |
| `File Report.pdf` | Write-up of the techniques |

## Usage

Requires Python 3. There are no dependencies.

### Round-trip test

Run this from the project directory. Pass the file name **without** the `.tex` extension:

```sh
python testEncoderDecoder.py bigbible
```

It prints the input, encoded and decoded sizes. It then prints `SUCCESS: Lossless compression` and the compression ratio, or an error if the decoded file differs from the input.

### Running the encoder and decoder separately

```sh
python encoder.py bigbible.tex     # writes bigbible.lz
python decoder.py bigbible.lz      # writes bigbible-decoded.tex
```

## Limitations

- The encoder expects a filename ending in a 3-letter extension (e.g. `.tex`). It strips the last 4 characters to build the output name.
- The LZ77 match search is naive, so compressing large files is slow.
- The Huffman header uses one byte per symbol, so it only supports characters up to code point 255 and up to 255 distinct symbols.
- The encoder appends a `-` marker at the end of the stream. The decoder strips a trailing `-`, so input files that end in `-` will not round-trip correctly.

## License

MIT. See [LICENSE](LICENSE).
