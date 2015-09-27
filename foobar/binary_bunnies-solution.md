---
layout: default
---
Strategy: look on the internet for "count ways to insert values into binary tree"

```
# the following link is very elucidating and contains the below pseudocode
# http://stackoverflow.com/questions/17119116/how-many-ways-can-you-insert-a-series-of-values-into-a-bst-to-form-a-specific-tr
function countInsertionOrderings(Tree t) {
    if t is null, return 1;
    otherwise:
       let m = numElementsIn(t.left);
       let n = numElementsIn(t.right);
       return choose(m + n, n) * 
              countInsertionOrderings(t.left) *
              countInsertionOrderings(t.right);
}
```

So, create the tree, then process it like that.

```java
package com.google.challenges;
import java.util.*;
import java.math.*;

// this code implements the algorithm here:
// http://stackoverflow.com/questions/17119116/how-many-ways-can-you-insert-a-series-of-values-into-a-bst-to-form-a-specific-tr

public class Answer {

  public static String answer(int[] seq) { 
    Tree t = null;
    for (int i=0; i < seq.length; i++) {
      Tree node = new Tree(seq[i]);
      if (t == null) t = node;
      else t.add(node);
    }

    return count_orderings(t).toString();
  }

  public static BigInteger count_orderings(Tree t) {
    if (t == null) return BigInteger.valueOf(1);

    int left_count = t.left == null ? 0 : t.left.count();
    int right_count = t.right == null ? 0 : t.right.count();

    return choose(left_count + right_count, right_count).
            multiply(count_orderings(t.left)).
            multiply(count_orderings(t.right));
  }

  private static Map<Integer, BigInteger> fact_cache = new HashMap<Integer, BigInteger>();
  public static BigInteger fact(int n) {
    if (! fact_cache.containsKey(n)) {
      BigInteger f = BigInteger.valueOf(1);
      for (int i = 1; i <= n; i++) {
        f = f.multiply(BigInteger.valueOf(i));
      }
      fact_cache.put(n, f);
    }
    return fact_cache.get(n);
  }

  private static Map<List<Integer>,BigInteger> choose_cache = new HashMap<List<Integer>,BigInteger>();
  public static BigInteger choose(int n, int k) {
    if (k > n) {
      return BigInteger.valueOf(0);
    }
    else {
      List<Integer> nk = Arrays.asList(n, k);
      if (!choose_cache.containsKey(nk)) {
        choose_cache.put(nk, fact(n).divide(fact(k).multiply(fact(n - k))));
      }
      return choose_cache.get(nk);
    }
  }
}

class Tree {
  public int value;
  public Tree left = null;
  public Tree right = null;

  public Tree(int v) {
    value = v;
  }

  public void add(Tree t) {
    if (t.value < value) {
      if (left == null) left = t;
      else left.add(t);
    }
    else {
      if (right == null) right = t;
      else right.add(t);
    }
  }

  public int count() {
    int c = 1;
    if (left != null) c = c + left.count();
    if (right != null) c = c + right.count();
    return c;
  }

}
```

