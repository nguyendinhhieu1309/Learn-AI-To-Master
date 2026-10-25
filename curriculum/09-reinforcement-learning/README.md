# Reinforcement Learning

## Outcomes

- Model an interactive task as states, actions, rewards, and transitions.
- Explain value functions and the exploration/exploitation trade-off.
- Train and evaluate a simple agent in a controlled environment.

## Prerequisites

Python, probability, basic ML, and comfort with iterative experiments. Deep learning is needed for deep RL methods.

## Suggested sequence

1. [MDPs and value learning](01-mdps-and-value-learning.md)
2. [Policy learning and evaluation](02-policy-learning-and-evaluation.md)
3. [Policy gradients and actor-critic practice](03-policy-gradients-actor-critic-and-practice.md)
4. [Algorithm worked lab](04-algorithm-worked-lab.md)

## Practice

- Solve a tiny finite MDP by hand and compare with dynamic programming.
- Compare random actions with a learned tabular Q policy.
- Run an agent with multiple seeds and report variability, not only the best run.
- Create an example where a poorly designed reward encourages an unintended behavior.
- Calculate one Q-learning update by hand, then compare learning curves across seeds.

## Checkpoint project

Train an agent in a small simulator, compare it with a meaningful baseline, evaluate across multiple seeds, and explain reward design, failure modes, and why the result may not transfer to the real world.
