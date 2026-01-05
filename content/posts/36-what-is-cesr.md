+++
draft = true
title = "What is CESR?"
slug = "what-is-cesr"
date = "2026-01-04"

[taxonomies]
tags=["cesr"]

[extra]
comment = true
+++

# What is CESR?

Asked at IIW 36, Spring 2023 in the SPAC talk

Composable Event Streaming Representation

Outline:

1. Add in your own cryptographic suite, your own set of CESR codes, and even your own CESR native data structures.
2. And do it in Rust
3. And do it in Python

This example shows a basic TLV encoding scheme in Python.

import io
import os


class TLVParser:
    def __init__(self, stream: io.BytesIO):
        self.stream: io.BytesIO = stream

    def read_type(self):
        return int.from_bytes(self.stream.read(1), byteorder='big')

    def read_length(self):
        return int.from_bytes(self.stream.read(2), byteorder='big')

    def read_value(self, length):
        value = self.stream.read(length)
        return value

    def sniff(self):
        current_pos = self.stream.tell()
        type_field = self.read_type()
        length_field = self.read_length()
        self.stream.seek(current_pos)  # Reset the stream position
        return type_field, length_field

    def parse(self):
        while True:
            try:
                type_field, length_field = self.sniff()
                current_pos = self.stream.tell()
                self.stream.seek(3, os.SEEK_CUR)  # eat first three bytes after sniff resets
                value = self.read_value(length_field)
                if not value:
                    print("End of stream")
                    break
                print(f"Type: {type_field}, Length: {length_field}")
                print(f"\tValue: {value}")
            except Exception as e:
                print("End of stream or error:", e)
                break

# Example usage

# Simulated TLV byte stream
tlv_stream = io.BytesIO(b'\x01\x00\x03abc\x02\x00\x04defg')

parser = TLVParser(tlv_stream)
print("\nStream contents:")
parser.parse()


Share this:
				Click to share on X (Opens in new window)
				X
			
				Click to share on Facebook (Opens in new window)
				Facebook
			Like Loading...

	
		Categories: [Uncategorized](https://kentbull.com/category/uncategorized/)

```python
import io
import os


class TLVParser:
    def __init__(self, stream: io.BytesIO):
        self.stream: io.BytesIO = stream

    def read_type(self):
        return int.from_bytes(self.stream.read(1), byteorder='big')

    def read_length(self):
        return int.from_bytes(self.stream.read(2), byteorder='big')

    def read_value(self, length):
        value = self.stream.read(length)
        return value

    def sniff(self):
        current_pos = self.stream.tell()
        type_field = self.read_type()
        length_field = self.read_length()
        self.stream.seek(current_pos)  # Reset the stream position
        return type_field, length_field

    def parse(self):
        while True:
            try:
                type_field, length_field = self.sniff()
                current_pos = self.stream.tell()
                self.stream.seek(3, os.SEEK_CUR)  # eat first three bytes after sniff resets
                value = self.read_value(length_field)
                if not value:
                    print("End of stream")
                    break
                print(f"Type: {type_field}, Length: {length_field}")
                print(f"\tValue: {value}")
            except Exception as e:
                print("End of stream or error:", e)
                break

# Example usage

# Simulated TLV byte stream
tlv_stream = io.BytesIO(b'\x01\x00\x03abc\x02\x00\x04defg')

parser = TLVParser(tlv_stream)
print("\nStream contents:")
parser.parse()
```

### Share this:

- Click to share on X (Opens in new window)
				X
- Click to share on Facebook (Opens in new window)
				Facebook
- 

Tagged as: [temp-publish](https://kentbull.com/tag/temp-publish/)

### Kent Bull
