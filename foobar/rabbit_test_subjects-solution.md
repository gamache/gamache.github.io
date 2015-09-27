---
layout: default
---
# Rabbit test subjects Solution

Strategy: sort each list, compare head of each list, return ratio

NB: They say "rounded to the nearest whole number", but they mean int(),
not round().

    def answer(y, x):
        smallest_elts = [float(sorted(y)[0]), float(sorted(x)[0])]
        smaller = min(smallest_elts)
        larger = max(smallest_elts)
        result = int((larger - smaller) * 100 / larger)
        return result


[Back to Foobar challenges](index.html)

