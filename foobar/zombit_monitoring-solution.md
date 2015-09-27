---
layout: default
---
# Zombit monitoring Solution

Strategy: set up a state machine, and turn the intervals into state
machine inputs.


    def answer(intervals):
        state_log = []
        for i in intervals:
            start_time, end_time = i
            state_log.append([start_time, "start"])
            state_log.append([end_time, "end"])
        sorted_state_log = sorted(state_log, key=lambda x: x[0])
        total_time = 0
        last_start = 0
        currently_started = 0
        for state in sorted_state_log:
            time, action = state
            if action == "start":
                if currently_started == 0:
                    last_start = time
                currently_started += 1
            elif action == "end":
                currently_started -= 1
                if currently_started == 0:
                    total_time += time - last_start
        return total_time



[Back to Foobar challenges](index.html)

