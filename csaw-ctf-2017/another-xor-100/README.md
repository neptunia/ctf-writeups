# Another Xor
**Category**: Crypto

100 Points

221 Solves


We do a known plaintext attack on `flag{`. We can also guess the keylength by abusing the fact that the md5 digest is hexadecimal only.
The key is repeated three times, but the third reptition (block) is the last few bytes of the checksum so we only consider the first two blocks.
Each time we get one byte of the key we get one byte from each of the two blocks. We basically will approach this by keeping track of a worklist of what we know so far.

```python
#!/usr/bin/env python

from collections import deque
from pwn import *
import sys, time

block2_offset = 0x43
key_offset = 0x26
digest_offset = 0x69

ciphertext = [0x27,0x4C,0x10,0x12,0x1A,0x01,0x00,0x49,0x5B,0x50,0x2D,0x55,0x1C,0x55,0x7F,0x0B,0x08,0x33,0x58,0x5D,0x1B,0x27,0x03,0x0B,0x52,0x28,0x04,0x0D,0x37,0x53,0x49,0x0A,0x1C,0x02,0x54,0x15,0x05,0x15,0x25,0x45,0x51,0x18,0x00,0x19,0x11,0x53,0x4A,0x00,0x52,0x56,0x0A,0x14,0x59,0x4F,0x0B,0x1E,0x49,0x0A,0x01,0x0C,0x45,0x14,0x41,0x1E,0x07,0x00,0x14,0x61,0x5A,0x18,0x1B,0x02,0x52,0x1B,0x58,0x03,0x05,0x17,0x00,0x02,0x07,0x4B,0x0A,0x1A,0x4C,0x41,0x4D,0x1F,0x1D,0x17,0x1D,0x00,0x15,0x1B,0x1D,0x0F,0x48,0x0E,0x49,0x1E,0x02,0x49,0x01,0x0C,0x15,0x00,0x50,0x11,0x5C,0x50,0x58,0x50,0x43,0x42,0x03,0x42,0x13,0x54,0x42,0x4C,0x11,0x50,0x43,0x0B,0x5E,0x09,0x4D,0x14,0x49,0x57,0x08,0x0D,0x44,0x44,0x25,0x46,0x43]
plaintext = [0x0] * len(ciphertext)

queue = deque()
for pair in enumerate(ordlist('flag{')):
    queue.append(pair)

while queue:
    offset, known = queue.popleft()

    if plaintext[offset] != 0x0: # we've already seen this
        continue

    print '\033[G' + ''.join(['.' if x == 0 else chr(x) for x in plaintext]),
    plaintext[offset] = known

    if offset >= digest_offset: # in digest
        pass # who fucking cares?
    elif offset < key_offset: # known plaintext
        queue.append((key_offset+offset, ciphertext[offset] ^ known))
    else: # known key
        key_index = offset - key_offset
        queue.append((key_index, ciphertext[key_index] ^ known)) # block 1
        queue.append((block2_offset + key_index, ciphertext[block2_offset + key_index] ^ known)) # block 2

print
print repr(plaintext)
print unordlist(plaintext)
print 'done'
```
