---
layout: default
---
# Undercover underground Solution

Strategy: look up "count ways to connect N nodes and K edges",
find recursive relation on MSE: http://math.stackexchange.com/questions/689526/how-many-connected-graphs-over-v-vertices-and-e-edges

"Definitely the last addendum" has a good formula.
Time limit exceeded, again and again.
Tried memoization, didn't do enough.

Went to Java, with BigInteger and everything memoized.  It worked.


    package com.google.challenges;
    import java.util.*;
    import java.math.*;

    public class Answer {
      // this code is an implementation of the recursive
      // algorithm here, under "Definitely the last addendum":
      // http://math.stackexchange.com/questions/689526/how-many-connected-graphs-over-v-vertices-and-e-edges

      public static String answer(int N, int K) {
        return big_answer(N, K).toString();
      }

      private static Map<List<Integer>,BigInteger> big_answer_cache = new HashMap<List<Integer>,BigInteger>();
      public static BigInteger big_answer(int N, int K) {
        if (K < (N-1)) {
          return BigInteger.valueOf(0);
        }
        else if (K == (N-1)) {
          return BigInteger.valueOf((int)Math.pow(N, N-2));
        }
        else {
          List<Integer> nk = Arrays.asList(N, K);
          if (!big_answer_cache.containsKey(nk)) {
            big_answer_cache.put(nk, get_answer_the_hard_way(N, K));
          }
          return big_answer_cache.get(nk);
        }
      }

      public static BigInteger get_answer_the_hard_way(int n, int k) {
        BigInteger m_sum = BigInteger.valueOf(0);
        for (int m = 0; m <= n-2; m++) {
          BigInteger p_sum = BigInteger.valueOf(0);
          for (int p = 0; p <= k; p++) {
            int choose_m = (n - 1 - m) * (n - 2 - m) / 2;
            p_sum = p_sum.add(choose(choose_m, p).multiply(big_answer(m+1, k-p)));
          }
          m_sum = m_sum.add(choose(n-1, m).multiply(p_sum));
        }
        return choose((n*(n-1)/2), k).subtract(m_sum);
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

Python, for comparison:

    import math

    def answer(N, K):
        str(count_paths(N, K))

    def memoize(fn):
        cache = {}
        def memoized_fn(*args):
            if args not in cache:
                cache[args] = fn(*args)
            return cache[args]
        return memoized_fn

    fact = memoize(math.factorial)

    def choose(n, k):
        if (n - k) < 0:
            return 0
        else:
            return fact(n) / (fact(k) * fact(n - k))

    choose = memoize(choose)

    def count_paths(n, k):
        if k < (n - 1):
            return 0
        if k == (n - 1):
            return n**(n-2)
        else:
            m_sum = 0
            for m in range(0, n-1):
                p_sum = 0
                for p in range(0, k+1):
                    choose_n = (n - 1 - m) * (n - 2 - m) / 2
                    p_sum += choose(choose_n, p) * count_paths(m+1, k-p)
                m_sum += choose(n-1, m) * p_sum

            return int(choose((n*(n-1)/2), k) - m_sum)

    count_paths = memoize(count_paths)



[Back to Foobar challenges](index.html)

