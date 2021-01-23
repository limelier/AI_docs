# state-based models for decision problems

## models

- describe a state (**representation**)
- identify special states & the problem space
- describe & validate transitions 
- specify a search strategy

## uninformed search strategies
these have no distinction between states
- random
- BFS (and Uniform Cost)
  - search states in order of distance from start
  - finds optimal solution
- DFS (and Iterative Deepening, IDS)
  - go as far as possible as fast as possible
  - go back from dead ends and try something else
- backtracking
- bidirectional 
  - start from both the start state and end state, meet in the middle


## informed search strategies
use heuristics to help distinguish between states
- greedy
  - best-first
- hill-climbing
  - select a local state that's at least as good as the current one
  - easily fooled by local maxima
- simulated annealing
  - has a chance to choose a worse state
  - chance gradually gets lower over time
- A* (and IDA*)
  - informed IDDFS
  - IDDFS explores nodes by distance from initial state
  - A* explores nodes by $d(S) + h(S)$ 
    - $d(S)$ - distance from initial state
    - $h(S)$ - estimated distance from final state
  - optimization: consistent heuristic
    - $h(A) \le h(A,B) + h(B)$ if $B$ reachable from $A$
  