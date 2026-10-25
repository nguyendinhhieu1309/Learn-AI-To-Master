# Markov Decision Processes and Value Learning

## Learning objectives

Describe an interactive task as an MDP, calculate returns, and understand how value estimates support action selection.

## 1. The agent-environment loop

At time `t`, an agent observes state `s_t`, chooses action `a_t`, receives reward `r_{t+1}`, and transitions to `s_{t+1}`. A Markov decision process (MDP) consists of states, actions, rewards, transition dynamics, and often a discount factor `gamma`. The Markov assumption says the state contains enough information to predict the next-state distribution given the action.

## 2. Return and reward design

The discounted return is `G_t = r_{t+1} + gamma*r_{t+2} + gamma^2*r_{t+3} + ...`. `gamma` near 0 prioritizes immediate reward; closer to 1 values future rewards more. Reward is a designed objective, not a guarantee that the agent learns the intent. Poor rewards can be exploited.

## 3. Value functions and Bellman idea

`V(s)` is expected future return from state `s` under a policy. `Q(s,a)` is expected return after taking action `a` in `s` and then following a policy. The Bellman equation expresses a value as immediate reward plus discounted next-state value:

$$
Q(s,a)=\mathbb{E}[r+\gamma\max_{a'}Q(s',a')].
$$

This recursive view lets an agent learn which actions lead to good future states.

## 4. Q-learning update

For a table of state-action values:

$$
Q(s,a) \leftarrow Q(s,a)+\alpha[r+\gamma\max_{a'}Q(s',a')-Q(s,a)].
$$

The bracketed term is the temporal-difference error: target minus current estimate. `alpha` is the learning rate. In a terminal state, the future value is zero.

Suppose current `Q=2`, reward `r=1`, best next `Q=4`, `gamma=0.9`, and `alpha=0.5`. The target is `1+0.9*4=4.6`; error is 2.6; updated Q is `2+0.5*2.6=3.3`.

## 5. Exploration and exploitation

Always choosing the current best action may miss better actions. Epsilon-greedy chooses a random action with probability `epsilon`, otherwise the best known action. Decay exploration carefully; evaluate the final policy without exploration noise when appropriate.

## Practice

Draw a small grid world with a goal and a penalty. Define states, actions, terminal conditions, and rewards. Identify at least one reward that could encourage unwanted behavior. State what the agent knows at each step.

## Common mistakes

- Forgetting to define terminal states or reward timing.
- Setting `gamma=1` in an endless task without checking whether returns stay finite.
- Using a state that hides relevant history, violating the Markov assumption.
- Assuming an increasing reward proves the agent behaves as intended.
