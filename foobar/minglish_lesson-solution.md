---
layout: default
---
Strategy: this is called topo sort -- look up algorithm, implement algorithm

```
import sets
def answer(words):
    before = {} # e.g. before["a"] = Set(); before["c"] == Set(["a", "b"])
    edges = sets.Set() # e.g. Set([("a", "b"), ("b", "c")])
    def add_edge(a, b):
        if a not in before: before[a] = sets.Set()
        if b not in before: before[b] = sets.Set()
        before[b].add(a)
        edges.add((a, b))

    prev_word = ""
    for word in words:
        for i in range(0, min(len(prev_word), len(word))):
            a = prev_word[i].lower()
            b = word[i].lower()
            if a != b:
                add_edge(a, b)
                break
        prev_word = word

    # this is topo sort.  from wikipedia, algorithm from Kahn (1962):
    """
    L <- Empty list that will contain the sorted elements
    S <- Set of all nodes with no incoming edges
    while S is non-empty do
        remove a node n from S
        add n to tail of L
        for each node m with an edge e from n to m do
            remove edge e from the graph
            if m has no other incoming edges then
                insert m into S
    if graph has edges then
        return error (graph has at least one cycle)
    else
        return L (a topologically sorted order)
    """

    sorted_letters = []
    while True:
        start_letters = [x for x in before if len(before[x]) == 0]
        if len(start_letters) == 0: break
        a = start_letters.pop()
        sorted_letters.append(a)
        del before[a]
        next_edges = filter(lambda ab: ab[0] == a, edges)
        for edge in next_edges:
            b = edge[1]
            edges.remove(edge)
            before[b].remove(a)
    return "".join(sorted_letters)
```

