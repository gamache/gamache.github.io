---
layout: default
---
# Minions board game Solution

Strategy: write a memoized recursive algorithm to walk through
game states.


    package com.google.challenges;
    import java.util.*;

    public class Answer {

      public static int answer(int t, int n) {
        return count_games(t, n, 0);
      }

      private static Map<List<Integer>, Integer> count_games_cache = new HashMap<List<Integer>, Integer>();

      private static int count_games(int t, int n, int pos) {
        List<Integer> cache_key = Arrays.asList(t, n, pos);

        if (count_games_cache.containsKey(cache_key)) {
          return count_games_cache.get(cache_key);
        }

        int spaces_to_right = n - pos - 1;

        if (spaces_to_right == 0) return 1;

        if (t < 1) return 0;

        int count = 0;

        // R move
        if (spaces_to_right > 0) {
          count = count + count_games(t-1, n, pos+1);
        }

        // L move
        if (pos > 0) {
          count = count + count_games(t-1, n, pos-1);
        }

        // S move is always technically possible
        count = count + count_games(t-1, n, pos);

        count = count % 123454321;
        count_games_cache.put(cache_key, count);
        return count;
      }
    }



[Back to Foobar challenges](index.html)

