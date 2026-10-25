# Reinforcement Learning Worked Lab: Bandits, Q-Learning, and Policy Gradients

Reinforcement learning (RL) is for sequential decisions where actions affect future states and rewards. The agent does not receive a correct action label for every state; it learns from interaction, which makes evaluation and safe exploration central.

## Learning objectives

- Formalize a task as a Markov decision process.
- Explain return, value functions, and Bellman equations.
- Calculate a tabular Q-learning update.
- Distinguish value-based learning from policy-gradient methods.
- Explain replay buffers, target networks, and PPO clipping.
- Evaluate agents across seeds and diagnose reward hacking.

## Prerequisites

Study [MDPs and value learning](01-mdps-and-value-learning.md), [policy learning and evaluation](02-policy-learning-and-evaluation.md), and [policy gradients](03-policy-gradients-actor-critic-and-practice.md).

## 1. Start with a multi-armed bandit

A bandit has actions with unknown reward distributions but no state transition. At each step $t$, the learner picks action $a_t$ and observes reward $r_t$.

Regret compares cumulative reward with the expected reward of always selecting the best action:

$$
R_T=T\mu^*-\sum_{t=1}^{T}\mathbb{E}[r_t],
\qquad
\mu^*=\max_a\mathbb{E}[r\mid a].
$$

An explore-exploit strategy must sometimes test uncertain actions to learn whether they are better. Pure exploitation can get stuck with an action that only looked good because of early random outcomes.

An $\epsilon$-greedy policy selects a random action with probability $\epsilon$ and otherwise selects the action with highest current estimated value. It is simple, but random exploration can be wasteful when some actions are clearly unsafe or expensive.

## 2. The Markov decision process

An MDP is commonly written $(\mathcal S,\mathcal A,P,R,\gamma)$:

- $\mathcal S$: states;
- $\mathcal A$: actions;
- $P(s'\mid s,a)$: transition probabilities;
- $R(s,a,s')$: reward;
- $\gamma\in[0,1)$: discount factor.

For a policy $\pi$, the return from time $t$ is:

$$
G_t=\sum_{k=0}^{\infty}\gamma^k r_{t+k+1}.
$$

The state-value function is $V^\pi(s)=\mathbb{E}_\pi[G_t\mid s_t=s]$. The action-value function is $Q^\pi(s,a)=\mathbb{E}_\pi[G_t\mid s_t=s,a_t=a]$.

The Bellman optimality relation for action values is:

$$
Q^*(s,a)=\mathbb{E}\left[r+\gamma\max_{a'}Q^*(s',a')\mid s,a\right].
$$

The maximum appears because this equation describes the optimal future policy, not necessarily the behavior policy used to collect the transition.

## 3. Tabular Q-learning update

Q-learning updates the value of the observed state-action pair:

$$
Q(s,a)\leftarrow Q(s,a)+\alpha\left[
r+\gamma\max_{a'}Q(s',a')-Q(s,a)
\right].
$$

The bracketed quantity is the temporal-difference error. For a terminal transition, set the bootstrapped next value to zero.

### Worked update

Suppose $Q(s,a)=2$, reward $r=1$, $\gamma=0.9$, the next state's largest action value is 4, and $\alpha=0.5$.

$$
\delta=1+0.9(4)-2=2.6,
\qquad
Q_{\mathrm{new}}(s,a)=2+0.5(2.6)=3.3.
$$

One update moves the estimate toward the sampled target 4.6. It does not prove that the estimate is correct; many visits and suitable exploration may be needed.

## 4. From tables to neural value functions

When the state space is large, a neural network may approximate $Q(s,a;\theta)$. A common Deep Q-Network (DQN) setup uses:

- a replay buffer to reuse decorrelated transitions;
- a separate target network to stabilize the bootstrapped target;
- an exploration policy such as $\epsilon$-greedy;
- a loss between predicted and target action values.

The target network is updated periodically or slowly. The replay buffer changes the training distribution; it does not create independent data when many transitions come from the same trajectory. A poor reward definition or unsafe data-collection policy remains a problem.

## 5. Policy gradients

Instead of estimating values for every action, a policy-gradient method directly parameterizes $\pi_\theta(a\mid s)$. The objective is expected return:

$$
J(\theta)=\mathbb{E}_{\tau\sim\pi_\theta}[G(\tau)].
$$

The score-function identity gives:

$$
\nabla_\theta J(\theta)
=\mathbb{E}\left[
\nabla_\theta\log\pi_\theta(a_t\mid s_t)\,\hat A_t
\right],
$$

where $\hat A_t$ estimates whether the action was better than a baseline expectation. A learned value function can reduce variance. It does not remove variance or guarantee improvement from every finite-sample update.

## 6. PPO and clipped updates

Proximal Policy Optimization (PPO) compares the new policy probability with the behavior policy probability:

$$
r_t(\theta)=\frac{\pi_\theta(a_t\mid s_t)}
{\pi_{\theta_{\mathrm{old}}}(a_t\mid s_t)}.
$$

Its clipped surrogate objective is:

$$
L^{\mathrm{clip}}(\theta)=
\mathbb{E}_t\left[
\min\left(
r_t(\theta)\hat A_t,\,
\operatorname{clip}(r_t(\theta),1-\epsilon,1+\epsilon)\hat A_t
\right)\right].
$$

Clipping discourages excessively large probability-ratio changes in this objective. It is not a hard trust-region guarantee, does not prevent all destructive policy changes, and does not replace evaluation.

## 7. Evaluation protocol

RL outcomes can vary substantially across random seeds and environment randomness. Report:

- environment version and episode horizon;
- reward definition and termination conditions;
- number of training interactions and update budget;
- reward curves with uncertainty across seeds;
- success rate and task-specific safety metrics;
- a random or scripted baseline;
- evaluation on separate episodes without training updates;
- sensitivity to altered initial states or dynamics.

Do not report only the best seed or the final episode. A moving average can hide instability; show the raw range or confidence interval too.

## 8. Reward hacking and constraints

The agent optimizes the reward it receives, not the informal goal in the project description. If reward omits a constraint, the agent may exploit that omission. Separate:

- objective terms that express the desired tradeoff;
- hard safety constraints that must never be violated;
- evaluation metrics that are not used for training;
- human review and emergency stop behavior.

In robotics or other physical systems, begin in a simulator, bound actions, test failure cases, and require a safe fallback. A high simulated reward is not sufficient evidence of real-world safety.

## Practice with answer checks

1. In the Q-learning update, what value is used for a terminal next state?
2. Why is the max over next actions in Q-learning even if the behavior policy explores?
3. What problem does a replay buffer aim to reduce, and what problem does it not solve?
4. Why use multiple training seeds?
5. Give one example of reward hacking in a navigation task.

### Answers

1. Zero future value, so the target is just the observed reward.
2. Q-learning is off-policy: its target evaluates the greedy next action even when behavior sometimes explores.
3. It reduces strong temporal correlation among sampled updates and allows transition reuse. It does not fix biased data, invalid rewards, or poor coverage.
4. Stochastic optimization and environment randomness can make performance highly variable; multiple seeds reveal stability.
5. A reward that only encourages reaching a coordinate may be exploited by passing through an unsafe region or colliding if collision cost is omitted or too weak.

## Mini-project

Implement a small tabular environment with at least four states and two actions. Define terminal conditions, compare random behavior with Q-learning, report the learning curve over at least ten seeds, and test one altered transition or reward setting. Explain which results are guaranteed by the design and which are only empirical.

## Common mistakes

- Treating reward as an unambiguous specification of the real goal.
- Comparing policies with different interaction budgets.
- Evaluating during training while exploration is still active and calling it deployment performance.
- Reporting one run without seed variation.
- Assuming clipping or a target network proves safety.
