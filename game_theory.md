<style>
table {
    margin: 0 auto;
}

td {
    border: 1px solid black;
    background: white;
    color: black;
    text-align: center;
    padding: 3px;
}

.agent1 {
    color: blue;
    text-align: right;
}

.agent2 {
    color: red;
}

.blank {
    border: none;
    background: transparent;
}
</style>

# game theory

- classically, searching involves one agent trying to reach its objective
- games involve searching in the presence of an opponent
- games are represented like search problems:
  - states: game board configurations
  - operators/actions: allowed moves
  - initial state: *current* game state
  - end state: winning game state

## sequential games

- players move one at a time
- have an heuristic $h(S)$ for how good a state is
- assuming a zero-sum game, we can use a single evaluation function for both players: $f$
  - the higher, the more advantageous to the computer
  - the lower (negative!), the more advantageous to the player
  - this evaluation function can be tricky to find

### minimax

- a minimax tree involves nodes in a tree where MIN and MAX levels alternate
- computer tries to maximize MAX levels, opponent tries to minimize MIN levels
- select a depth limit and an eval function, and generate the tree up to $n$ deep
- evaluate the leaves, propagate their value up 
  - select minimum value for MIN
  - select maximum value for MAX
- for trees with chance-nodes (e.g.: dice throws), use the *expected values* for the chance-nodes
  - chance nodes before a MIN layer
    $$expectimin(A) = \sum_i(P(d_i) * minvalue(i))$$
  - chance nodes before a MAX layer
    $$expectimax(A) = \sum_i(P(d_i) * maxvalue(i))$$
  - basically, weighted sums

### alpha-beta pruning

- an optimization for minimax
- cancel generation for parts of the tree that aren't needed
- the idea is to keep an estimate for each node based on the currently explored nodes below
  - $\alpha$ for MAX nodes, updated as descendants are explored
  - $\beta$ for MIN nodes, updated as descendants are explored
  - if `parent.alpha >= self.beta`, stop exploring `self`, since `self.beta` only goes down
  - if `parent.beta <= self.alpha`, stop exploring `self`, since `self.alpha` only goes up
- for trees deeper than 4 levels, alpha-beta pruning applies to deeper levels too
  - look up the `alpha` or `beta` of not only the node above, but the nodes 3, 5, etc. above too - all of the `alpha`s or `beta`s on the way to the root
- other optimizations include:
  - *forward-pruning* - ignore sub-trees if they don't make sense to explore
    - too similar to ones encountered before?
    - have seemingly irrational moves?
    - only recommended at big depths, not near the root
  - searching with a time limit: return the best move *so far* when the time runs out
  
### monte carlo tree search

- apply the following algorithm:

1. **selection**:
   - all nodes have a value: the ratio of wins passing through that node vs. total times passed through that node ($wins / passes$)
   - select the max ratio on each level to reach a node where statistics don't exist for all children
   - *upper confidence bound* (UCB1) selection maximizes the following expression instead: 
    $$\frac{w_i}{n_i} + \sqrt{\frac{2\ln{n}}{n_i}}$$
      *$w_i$ = wins for child $i$, $n_i$ = passes for child $i$, $n$ = passes for self*
   - selection can pick a node that's already a terminal state
     - if it's a loss, give it a very small value to prevent its choice next time
     - if it's a win, give it a very large value, and give its parent a very small value
2. **expansion**:
   - applied when selection cannot continue
   - select a random unvisited leaf node and add a new record for each of its sons (0/0)
   - nodes with 0/0 will be selected first every time using UCB1, as they have infinite value  

3. **simulation**:
   - after expansion, start simulation
   - do random moves until you get to a terminal state (win/loss)
   - this kind of simulation is also called *rollout* or *playout*
   - sometimes, instead of a completely random search, you can use weighting heuristics to choose "better" moves

4. **actualization** (or *retro-propagation*)
   - after simulation, increment passes (and wins) for all visited nodes
   - a win is incremented only on the nodes corresponding to the winning player
   - don't update down, just up from the node added in the expansion step to the root

- after repatedly applying the above algorithm, choose the move with the most passes, as its value is the best estimated
  - since it was explored most, its value must be great too
- after the computer and the opponent move, the corresponding sub-tree can be reused as initial values for the next iteration

### MCTS vs minimax
- MCTS doesn't need heuristics
- MCTS is asymmetrical: exploration converges towards better moves
- MCTS is *anytime*: it can produce an estimate of the best move at any time

## strategy games

- different from sequential games
- these games represent strategic, *simultaneous* interactions between rational agents
  - these agents are assumed to pick their actions to maximize their winnings

- elements:
  - at least 2 players or **agents**
  - each agent has a number of different **strategies** to use
  - the chosen strategies of each agent determine the **outcome** of the game
  - for every outcome, there is a numeric **payoff** for each player
- these elements can be represented by a *payoff matrix*, as seen below

### famous example: prisoner's dillema
- 2 prisoners (suspects?)
- both have the same available strategies: confess to the crime, or deny involvement
- the agents choose their actions simultaneously, without knowing what the other agent is going to choose
- outcome: years of prison time
- payoff: less years $\rightarrow$ bigger payoff
- example payoff matrix:
<table>
    <tr>
        <td class="blank" ></td>
        <td class="agent2">deny</td>
        <td class="agent2">confess</td>
    </tr>
    <tr>
        <td class="agent1">deny</td>
        <td>-1, -1</td>
        <td>-5, 0</td>
    </tr>
    <tr>
        <td class="agent1">confess</td>
        <td>0, -5</td>
        <td>-3, -3</td>
    </tr>
</table>

- the first payoff in each cell is for *agent 1* (blue) and the second is for *agent 2* (red)

### zero-sum games
- for any result of the game, the sum of agent payouts will be 0. example:

<table>
    <tr>
        <td class="blank" ></td>
        <td class="agent2">C</td>
        <td class="agent2">D</td>
    </tr>
    <tr>
        <td class="agent1">A</td>
        <td>2, -2</td>
        <td>-1, 1</td>
    </tr>
    <tr>
        <td class="agent1">B</td>
        <td>1, -1</td>
        <td>-3, 3</td>
    </tr>
</table>

- two-player zero-sum games can be represented by a simplified payout matrix:
<table>
    <tr>
        <td class="blank" ></td>
        <td class="agent2">C</td>
        <td class="agent2">D</td>
    </tr>
    <tr>
        <td class="agent1">A</td>
        <td>2</td>
        <td>-1</td>
    </tr>
    <tr>
        <td class="agent1">B</td>
        <td>1</td>
        <td>-3</td>
    </tr>
</table>

- the represented winnings are for the first agent (blue), and they are reversed for the second one

### domination
- a strategy $S$ dominates a strategy $T$ of the same agent if any result of $S$ is at least as good as the corresponding result in $T$
- a rational agent should never play a dominated strategy
- if every agent has a dominating strategy and plays it, their combination and the respective payoffs is called the game's **dominant strategy equilibrium**
- example:
<table>
    <tr>
        <td class="blank" ></td>
        <td class="agent2">C</td>
        <td class="agent2">D</td>
    </tr>
    <tr>
        <td class="agent1">A</td>
        <td>1, 3</td>
        <td>4, 2</td>
    </tr>
    <tr>
        <td class="agent1">B</td>
        <td>2, 4</td>
        <td>7, 1</td>
    </tr>
</table>

- $B$ dominates $A$ for agent 1, and $C$ dominates $D$ for agent 2
- this means that $(B, C)$ will be the dominant strategy equilibrium
- not all dominations will be apparent at first
  - the **principle of higher order dominance**
  - cross out the dominated strategies
  - excluding the crossed-out strategies, more dominations will appear
  - repeat until equilibrium found
  - example: 
    - $D$ dominates $E$, cross out $E$
    - $B$ now dominates $A$, cross out $A$
    - $D$ now dominates $C$, cross out $C$
    - what's left is the equilibrium, $(B, D)$
<table>
    <tr>
        <td class="blank" ></td>
        <td class="agent2">C</td>
        <td class="agent2">D</td>
        <td class="agent2">E</td>
    </tr>
    <tr>
        <td class="agent1">A</td>
        <td>1, 1</td>
        <td>2, 0</td>
        <td>3, -1</td>
    </tr>
    <tr>
        <td class="agent1">B</td>
        <td>2, 1</td>
        <td>4, 3</td>
        <td>2, 0</td>
    </tr>
</table>


### pure Nash equilibrium

- a combination of strategies is a **Nash equilibrium** if each player maximizes their winnings, given other player's strategies
- it identifies stable strategy combinations, in the sense that switching strategies will not yield better results unless the other agents do the same
- idk what the $u_i(s_i^*, s_{-i}^*) \ge u_i(s_i, s_{-i}^*)$ stuff is about but thats a nash equilibrium
  - it's strict if $\gt$ instead
- to find pure nash equilibriums:
  - write $\{$ before the payoff for the first player if it's the best on the column
  - write $\}$ after the payoff for the second player if it's the best on the line
  - pure nash equilibriums will be between curly braces
  - example:
<table>
    <tr>
        <td class="blank" ></td>
        <td class="agent2">deny</td>
        <td class="agent2">confess</td>
    </tr>
    <tr>
        <td class="agent1">deny</td>
        <td>-1, -1</td>
        <td>-5, 0 }</td>
    </tr>
    <tr>
        <td class="agent1">confess</td>
        <td>{ 0, -5</td>
        <td>{ -3, -3 }</td>
    </tr>
</table>

### mixed Nash equilibrium

- sometimes a pure nash equilibrium does not exist
- example: the welfare game (Government, Pauper)

<table>
    <tr>
        <td class="blank" ></td>
        <td class="agent2">do work</td>
        <td class="agent2">be idle</td>
    </tr>
    <tr>
        <td class="agent1">aid</td>
        <td>{ 3, 2</td>
        <td>-1, 3 }</td>
    </tr>
    <tr>
        <td class="agent1">don't aid</td>
        <td>-1, 1 }</td>
        <td>{ 0, 0</td>
    </tr>
</table>

- none of the cells work as a NE:
  - (aid, do work) - pauper prefers *be idle*
  - (aid, be idle) - govt prefers *don't aid*
  - (don't aid, be idle) - pauper prefers *do work*
  - (don't aid, do work) - govt prefers *aid*

- mixed strategies:
  - in a pure strategy, player $i$ picks strategy $s_{ij}$ from set $S_i$
  - in a mixed strategy, player $i$ picks strategy $s_{ij}$ with probability $p_{ij}$
  - every pure strategy is a mixed strategy too, just with probabilities $p_{ij} = 1$ and $p_{ij'} = 0, \forall j' \ne j$
  - a finite game always has either a *pure* or a *mixed* nash equilibrium
  - payoff in mixed strategies is called *expected payoff* - the weighted average payoff of each strategy
  - how do you interpret mixed strategies?
    - games where you can employ multiple strategies simultaneously (betting on more than one horse)
    - multiple instances of the same game (do this x% of the time)
    - infinite instances of the same game
    - estimating a player's decision for a single game
- the **oddment method**
  - a simple way to compute mixed NE
  - N/A if the game has a pure NE 

<table>
    <tr>
        <td class="blank" ></td>
        <td class="agent2">do work</td>
        <td class="agent2">be idle</td>
    </tr>
    <tr>
        <td class="agent1">aid</td>
        <td>3, 2</td>
        <td>-1, 3</td>
    </tr>
    <tr>
        <td class="agent1">don't aid</td>
        <td>-1, 1</td>
        <td>0, 0</td>
    </tr>
</table>

- government payoffs: $
  \begin{bmatrix}
  3 & -1 \\
  -1 & 0 \\
  \end{bmatrix}
  $ - used for pauper strategy
  - $3-(-1)=4 \Rightarrow$ *work* with probability $\frac{4}{|-1|+4} = 0.8$
  - $-1-0=-1 \Rightarrow$ *be idle* with probability $\frac{|-1|}{|-1|+4} = 0.2$
  - if the pauper chooses to *work* with prob. $0.2$, the govt will get the same payoff from *aid* and *don't aid*

- pauper payoffs: $
  \begin{bmatrix}
  2 & 3 \\
  1 & 0 \\
  \end{bmatrix}
  $ - used for govt strategy
  - $2-3 = -1 \Rightarrow$ *aid with* probability $\frac{|-1|}{|-1|+1} = 0.5$
  - $1-0 = 1 \Rightarrow$ *don't aid* with probability $\frac{1}{|-1|+1} = 0.5$
  - if the govt chooses *aid* with prob 0.5, the pauper will get the same payoff from *do work* or *be idle*
- for the probabilities $(0.5, 0.5)$ and $(0.2, 0.8)$, both the govt and the pauper have equal expected payoffs for both actions, which allows a **nash equilibrium**
- more intuitive method: equations
  - let govt have probabilities $(x, 1-x)$ and pauper have $(y, 1-y)$
  - attempt to produce equal payouts between all available strategies for the *other player*
  - for govt: 
  $$
  \begin{aligned}
  2x + 1(1-x) &= 3x + 0(1-x) \\ 
  (-1)x+1(1-x) &= 0 \\ 
  1-2x &= 0 \\
  2x &= 1 \\
  x &= 0.5
  \end{aligned}
  $$
  - for pauper:
  $$
  \begin{aligned}
  3y + (-1)(1-y) &= (-1)y + 0(1-y) \\
  4y + (-1)(1-y) &= 0 \\
  5y - 1 &= 0 \\
  5y &= 1 \\
  y &= 0.2
  \end{aligned}
  $$
- if a player leaves the equilibrium strategy, the opponent can take advantage to gain more than they would at equilibrium

### pareto optimality
- an outcome is **Pareto optimal** if it cannot be improved for anyone without harming someone
- an outcome $O_1$ **dominates** $O_2$ iff:
  - $\forall i, O_1(i) \ge O_2(i)$
  - $\exists i, O_1(i) \gt O_2(i)$
- the non-dominated outcomes are pareto optimal
- for a strategy game, a *pareto optimal state* means the players do not have the motivation to deviate *in a coalition*
- example: prisoner's dillema
<table>
    <tr>
        <td class="blank" ></td>
        <td class="agent2">deny</td>
        <td class="agent2">confess</td>
    </tr>
    <tr>
        <td class="agent1">deny</td>
        <td>-1, -1</td>
        <td>-5, 0</td>
    </tr>
    <tr>
        <td class="agent1">confess</td>
        <td>0, -5</td>
        <td>-3, -3</td>
    </tr>
</table>

- $(-1, -1)$ dominates $(-3, -3)$, players will want to deviate from the latter to the former
- $(0, -5)$ and $(-5, 0)$ are not dominated, one player cannot deviate from them without causing the other a disadvantage
- all situations except (*confess*, *confess*) are pareto optimal: no prisoner can do less prison time without the other doing more

## two-player cooperative games
- previously: agents were rational and selfish
- by cooperating, agents can obtain a greater total payoff (maximum sum of payoffs)
- example: blue (Renault) and red (Peugeot)

<table>
    <tr>
        <td class="blank" ></td>
        <td class="agent2">F</td>
        <td class="agent2">M</td>
    </tr>
    <tr>
        <td class="agent1">F</td>
        <td>-10, -40</td>
        <td>40, 10</td>
    </tr>
    <tr>
        <td class="agent1">M</td>
        <td>10, 40</td>
        <td>-40, -10</td>
    </tr>
</table>

- **sum matrix**: $R + P = \begin{bmatrix}-50 & 50 \\ 50 & -50 \end{bmatrix}$
  - total payoff: $\max(R+P) = 50$
- **threat matrix**: $R - P = \begin{bmatrix}30 & 30 \\ -30 & -30\end{bmatrix}$
  - Renault will *always* gain more than Peugeot by picking $F$
  - i have *no* idea what a "threat differential" is but
  - i assume that it's the value of the dominant strategy equilibrium's cell for treating the threat matrix as a 0-sum game
- play whatever strategy gives the right total payoff, settle the difference after the game if needed
  - either play (F, M) and have nothing to settle (50 total, Renault gets 30 more than Peugeot)
  - or play (M, F) and settle after (50 total, Peugeot gets 30 more than Renault, P pays R 30 units)

## N-player cooperative games
### representing the game in characteristic form
- $n$ agents, $P_1, P_2, ..., P_n$
- the **grand coalition** is the entire set of agents, $G = {P_1, ..., P_n}$
- a **coalition** is a non-empty subset of $G$ (a team of one or more players)
- each coalition tries to maximize its payoff
- the *value* of each coalition (the $v$ characteristic function) is its max payoff
- **superadditive game**: $v(S \cup T) \ge v(S) + v(T)$, where $S, T$ coalitions with no agents in common
- an **imputation** is a set of payoffs $(x_1, ..., x_n)$ s.t.:
  - the sum of payoffs is equal to the value of the grand coalition
  $$\sum_{i=1}^nxi = v(G)$$
  - each agent gets a payoff at least as good as if it was working alone
  $$x_i \ge v(\{P_i\})$$
  - an imputation is both efficient and individually rational
- example: 
  - agents: $P_1$, $P_2$, $P_3$
  - each chooses heads ($H$) or tails ($T$)
  - if all agents play the same side, nothing happens
  - if two agents pick the same thing and the third picks different, he pays both 1
  - $v({P_1, P_2, P_3} = 0)$ - zero-sum game
  - the coalition $S = {P_2, P_3}$ forms
  - the counter-coalition is $S^c = {P_1}$
  - thus, zero-sum game with the matrix:
<table>
    <tr>
        <td class="blank" ></td>
        <td class="agent2">HH</td>
        <td class="agent2">HT</td>
        <td class="agent2">TH</td>
        <td class="agent2">TT</td>
    </tr>
    <tr>
        <td class="agent1">H</td>
        <td>0</td>
        <td>1</td>
        <td>1</td>
        <td>-2</td>
    </tr>
    <tr>
        <td class="agent1">T</td>
        <td>-2</td>
        <td>1</td>
        <td>1</td>
        <td>0</td>
    </tr>
</table>

- continuing:
  - $S$ will never choose $HT$ or $TH$ (these are dominated)
  - the value of the game is a $-1$ mixed equilibrium
    - $S^c$ expects to lose 1
    - $S$ expects to gain 1
  - due to the symmetry of the game, the value function is:
    - $v(\{P_1\}) = v(\{P_2\}) = v(\{P_3\}) = -1$
    - $v(\{P_1, P_2\}) = v(\{P_2, P_3\}) = v(\{P_1, P_3\}) = 1$
    - $v(\{P_1, P_2, P_3\}) = 0$
  - imputations: any $(x_1, x_2, x_3)$, as long as $x_i \ge -1$ and $x_1 + x_2 + x_3 = 0$

### the core
the core of an $n$-agent game is the set of **non-dominated imputations**

- the core of a game with $v$ is the set of imputations $x = (x_1, x_2, ..., x_n)$ s.t. for any coalition $S = \{P_{i1}, P_{i2}, ..., P_{im}\}$ we have $x_{i1} + ... + x_{im} \ge v(S)$
  - any coalition ends up getting at least as much as its value
- any imputation in the core can be viewed as a game solution
- the core is stable
- if an imputation is not in the core, there's at least one coalition getting less than its value - these agents will prefer another imputation
- zero-sum games have an empty core (the game is unstable)

### the Shapley value
- the core offers a set of solutions for a game
- what if the game doesn't have a core?
- probably the friendliest expression for the shapley value:
$$\phi(i) = \frac{1}{n!}\sum_{i\in S}(|S|-1)!(n - |S|)!(v(S) - v(S-\{i\}))$$
- the shapley value always exists, is unique, and is feasible (the sum of the payoffs is maximum)
- it may not belong in the game's core
  - if it does not, it is *unstable*

### convex games
a game is **convex** if its characteristic function ($v$) is *supermodular*:
$$v(S \cup \{i\}) - v(S) \le v(T \cup {i}) - v(T), \forall S \subseteq T \subseteq N - \{i\}, \forall i \in N$$
- in a nutshell, apparently, the motivation to join a coalition (the possible payoff) grows as the coalition grows
- any convex game is *superadditive*
- the core of a convex game is always *non-empty*, and the shapley value belongs to the core, at its *center of gravity*

### conclusions
- a finite strategic game always has at least one pure or mixed nash equilibrium
- an outcome is pareto optimal if it can't get better for someone without getting worse for someone else
- in a nash equilibrium, agents aren't motivated to deviate individually
- in a pareto optimal state, agents aren't motivated to deviate as a coalition
- through cooperation, agents can obtain a higher payoff
  - but how do you split the payoff?
- the core of an n-player game is the set of non-dominated inputations
  - and it's stable, i guess
- the basic idea of the shapley value: each agent should get a payoff corresponding to its marginal contribution to the possible coalitions
  - sure, it might be impossible to gain something without agent 1 on the team, but he won't get anything alone either, so...
- this course is a goddamn mess