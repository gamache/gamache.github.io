---
layout: default
---
# Peculiar balance Solution

Strategy: this is called balanced ternary and has been known for
centuries.  There's an algorithm for it.


    def answer(x):
        # this is basically pasted from Rosetta Code
        if x == 0:
            return []
        elif (x%3) == 0:
            return ["-"] + answer(x // 3)
        elif (x%3) == 1:
            return ["R"] + answer(x // 3)
        elif (x%3) == 2:
            return ["L"] + answer((x+1) // 3)



[Back to Foobar challenges]({{ "/2015/08/01/google-foobar.html" | prepend: site.baseurl }})

