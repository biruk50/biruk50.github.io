---
layout: post
title: "Making Reinforcement Learning Explicitly Learn from State Changes"
date: 2026-10-19
categories: [reinforcement-learning]
tags: [reinforcement learning, PPO, neural networks]
excerpt: "An experiment comparing state-difference inputs and a separate-branches model with standard PPO observations."
---

What happens when we feed a reinforcement learning agent not only its current state, but also information about how that state is changing?

![Animated visualization accompanying the reinforcement learning experiment.](https://miro.medium.com/v2/resize:fit:1200/format:webp/1*wYpdJzuSZF5fq6YCFFBX-Q.gif)

## Introduction

In reinforcement learning (RL), an agent repeatedly observes its environment, selects an action, and receives a reward. Its policy learns to choose actions that maximize expected cumulative reward.

A typical implementation feeds the **current observation directly** into a neural network. In continuous-control tasks, that observation may contain position, velocity, angles, angular velocities, or other measurements describing the environment.

However, some information about how the state changes is **already included in conventional observations.** For example, velocity describes how position changes over time. Many environments calculate or simulate velocity and include it in the observation.

To investigate this, we use environments from [Gymnasium](https://gymnasium.farama.org/) and train agents using Proximal Policy Optimization (PPO), with the default policy architecture and hyperparameters provided by Stable-Baselines3 as our baseline.

First approach: Feeding only state changes to the network
---------------------------------------------------------

This approach performed very poorly across the environments we tested.

One possible explanation is that many tasks depend on the absolute values of state variables, not just their changes. For example, knowing that a robot’s position has changed does not necessarily tell the agent where the robot currently is. Removing the original observation discards this potentially important information.

Second approach: Combining the state and its changes
----------------------------------------------------

Instead of replacing the current observation, we concatenate it with its difference from the previous observation.

For example, if an environment normally provides four observation values, the network receives eight values: the four current observations followed by the four corresponding differences.

![Results on the Hopper environment.](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*FbpzRuG9AxH-ycxU9qSCww.png)

![Results on the Swimmer environment.](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*cZvtUzbPQ060R2DGCQfbIw.png)

In the experiments, the combined-input approach performed **slightly better** in some environments and appeared to reach higher rewards somewhat faster. One possible explanation is that providing state differences makes information about recent changes more immediately accessible to the network, rather than requiring it to learn useful representations of those changes from the original observations alone.

The approach was **less successful** in more challenging environments, including **Lunar Lander** and **Walker2d**. Simply concatenating the two inputs did not consistently improve learning.

Third approach: A separate-branches model
-----------------------------------------

In this approach, the current observation and the observation difference are processed independently by two neural-network branches. Their learned features are then concatenated and passed to PPO’s policy and value networks.

![Diagram of the separate-branches model.](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*CTiZ7lUIQYt7s50Obei_EA.png)

The current-state branch receives a larger share of the feature-processing capacity, while the difference branch receives a smaller share. I used a **75/25 allocation** as my design choice, favoring the current-state branch.

The intention is to let each branch specialize in a different type of information before combining their representations. The current-state branch can focus on the present configuration of the environment, while the difference branch can learn representations of recent changes.

We also aim to keep the overall model capacity comparable to that of the baseline, so that any improvement is less likely to be explained simply by having a larger network.

![Result of the Lunar Lander experiment](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*kyhqsnS-1DrZqQGE4R1GJw.png)

The results show a modest advantage for the separate-branches model in this experiment. The recorded episodes show a more noticeable difference in performance.

![Original PPO behavior.](https://miro.medium.com/v2/resize:fit:1200/format:webp/1*1CHSrWbOwkSgPNmU9vA1dA.gif)

![Behavior using the separate-branches method.](https://miro.medium.com/v2/resize:fit:1200/format:webp/1*tGlLxoUP4uAXnZEmp__4ew.gif)

![Result on the Walker2d environment.](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*khUMsayWvIJFYtn4S5MxDA.png)

This approach also led to better performance and smoother movement in the bipedal Walker2d environment.

Conclusion
----------

This experiment investigates a simple change to the input-processing pipeline of a reinforcement learning agent: instead of feeding PPO only the current observation, we also provide the difference between the current and previous observations.

These experiments do **not show** that this **architecture is universally better than standard PPO**, nor that it will outperform improvements to the reinforcement learning algorithm itself. Rather, they suggest that **how observations are represented and processed may influence learning and the resulting behavior**.

There is also a potential trade-off: allocating part of a fixed model capacity to the difference branch leaves less capacity for processing the current state. In a more demanding environment, such as Walker2d, increasing the overall network size may help compensate for this limitation. That possibility needs to be tested while controlling for parameter count.

The central question is not whether state differences should always be included, but **when explicitly representing state changes helps an RL agent learn more effectively — and what architecture makes the best use of that information.**

_To find the full code/results click_ [_here_](https://github.com/biruk50/RL/tree/main/CHANGES_AS_INPUT)_._

_If you have any additional insights or corrections, please share them in the comments. Thank you for reading!_