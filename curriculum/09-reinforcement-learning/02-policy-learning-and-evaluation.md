# Policy Learning and Evaluation

## Learning objectives

Understand the difference between value-based and policy-based methods, evaluate stochastic training fairly, and recognize simulation-to-reality limits.

## 1. From value to policy

A **policy** `pi(a|s)` describes which action to take in a state. Value-based methods learn `Q(s,a)` then select high-value actions. Policy-based methods directly optimize a policy, which is useful for stochastic or continuous actions. Actor-critic methods combine a policy (actor) with a value estimate (critic).

## 2. Policy gradient intuition

The policy gradient increases the probability of actions that led to better-than-expected outcomes and decreases the probability of worse ones. A baseline such as a state value can reduce gradient variance without changing the ideal direction. Training is noisy because data depend on the current policy and environment.

Deep Q-networks use function approximation for large state spaces, with techniques such as replay buffers and target networks to stabilize learning. These techniques help but do not make training deterministic or universally stable.

## 3. Evaluate multiple runs

Use multiple random seeds and report the distribution (for example, median and spread), not just the best run. Compare with random, heuristic, or simple programmed baselines. Plot reward and success rate; inspect trajectories, constraint violations, and failure modes. Fix environment versions and settings.

## 4. Reward hacking and safety

An agent optimizes the reward it receives, including accidental loopholes. Test unusual states, delayed effects, and edge conditions. Apply constraints or shields where safety is required; do not rely only on reward penalties. Offline RL learns from logged actions and must handle distribution mismatch—an offline dataset may not contain safe or successful actions for every situation.

## 5. Sim-to-real

A simulated policy may fail on a real robot because physics, sensors, timing, materials, and disturbances differ. Domain randomization can expose the policy to variation but does not prove safety. Validate in stages: offline, simulation, controlled hardware tests, then wider operation with limits and emergency stops.

## Practice

Train a policy in a small simulator with five or more seeds. Compare against a baseline. Report average and variability, show example successful and failed episodes, and test one changed environment condition. Explain what evidence would still be needed before real deployment.

## Common mistakes

- Reporting only the luckiest run.
- Confusing reward with the actual objective or safety requirement.
- Assuming simulation success transfers to physical hardware.
- Tuning to a fixed benchmark while ignoring variation and constraints.
