---
layout: default
---
# Hash it out Solution

Strategy: since the input space is only 64K large, write a generator for the
input -> output mappings, then invert it to output -> input.


    def answer(digest):
        digests = {}
        for x_minus_one in range(0, 256):
            for x in range(0, 256):
                d = ((129 * x) ^ x_minus_one) % 256
                if d not in digests:
                    digests[d] = []
                digests[d].append([x_minus_one, x])
        last_byte = 0
        output_bytes = []
        for byte in digest:
            for byte_pair in digests[byte]:
                lb, b = byte_pair
                if last_byte == lb:
                    output_bytes.append(b)
                    last_byte = b
                    break
        return output_bytes



[Back to Foobar challenges]({{ "/2015/08/01/google-foobar.html" | prepend: site.baseurl }})

