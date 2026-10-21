---
layout: distill
title: "Policy, World Model Code Generation"
description: "Generative policies, world model rollouts, and using LLMs for robotic code generation (ProgPrompt, WorldCoder)."
date: 2026-10-12
future: true
htmlwidgets: true
ready: true

# Reporter team authors
authors:
  - name: "Lainey Gordon"
  - name: "William Smith"
  - name: "Wendy Zheng"
  - name: "Andrew Parkinson"

bibliography: 2026-10-12-policy-world-model-code-generation.bib

toc:
  - name: "Introduction"
  - name: "ProgPrompt"
  - name: "WorldCoder"
  - name: "Applications and Limitations"
    subsections:
      - name: "Applications"
      - name: "Limitations"
  - name: "Questions and Answers"
    subsections:
      - name: "ProgPrompt Questions"
      - name: "WorldCoder Questions"
---

## Introduction {#introduction}

This lecture report covers the **Policy, World Model Code Generation** session in *Learning for Interactive Robots (CS 6501, Fall 2026)* at the University of Virginia.

> **Topic Overview**: Generative policies, world model rollouts, and using LLMs for robotic code generation (ProgPrompt, WorldCoder).

---

## ProgPrompt {#progprompt}

---

## WorldCoder {#worldcoder}
# WorldCoder: Summary

WorldCoder uses an LLM to generate a world model in the form of Python code, then uses that world model to create a plan.

## Background

People can pick up a new game or device in a handful of tries. We have strong priors from similar things we've seen and are exceptionally good at transferring skills to new goals. This is akin to learning a **world model**: a transition function that predicts the next state given the current state and an action. LLMs have a ton of knowledge about the world baked into their weights.

### Limitations of existing approaches

- **Deep model-based RL:** Has weak priors and high sample complexity. With no prior knowledge, it must try countless actions until something sticks, and the trained network is a black box that is difficult to inspect.
- **LLM agents:** Have strong priors and don't need environment interaction to start, but they never update a world model. Each action also requires a separate LLM call, so cost grows with episode length.
- **WorldCoder:** Has strong priors and low sample complexity. It learns a world model that is inherently readable, and once the model is learned it needs at most one LLM query per new goal.

LLMs know a lot about the world but can't efficiently search or plan. WorldCoder takes advantage of that knowledge without relying on the LLM for what it's bad at.

## Contextual MDP

A standard MDP has state, action, transition, and reward: ⟨S, A, T, R⟩. WorldCoder uses a **contextual MDP**, which has context, state, action, and model: ⟨C, S, A, M⟩. The context *c* is the natural-language goal, and M(c) gives an MDP for it. The transition function is shared across goals and doesn't depend on context, while the reward changes with the goal.

The LLM builds two Python programs:

- **T̂**: the learned transition function
- **R̂**: the learned reward model, which also returns a flag indicating whether the goal has been reached

## Constraints

- **φ₁ (Consistency):** T̂ and R̂ must match everything the agent has seen in the past.
- **φ₂ (Optimism):** The agent believes the goal is reachable, meaning there is a sequence of actions that reaches the goal with positive reward.

When either constraint is violated, the world model is updated. Optimism drives exploration, and consistency acts as guardrails.

## Pipeline

1. Receive a natural-language goal *c* and record the initial state.
2. At each step, if the model violates φ₁ or φ₂, call the LLM to refine the code.
3. Act: take a random action with 5% probability, otherwise plan with the current model.
4. Store the experience (s, a, r, s′, c, d) in the replay buffer and repeat.

The LLM is good at debugging, so showing it the previous code and the transitions it fails to explain lets it generate a better function. Because a new world model can be built by editing the previous one, knowledge transfers across environments.

## Experiments

### Sokoban

- The agent builds a working model from its first ~50 actions and generalizes from 2-box to 3-box levels.
- ReAct solves only ~15% of basic levels, and deep RL needs over 1M steps for 2-box levels.
- WorldCoder needs about 400k tokens at the start, then no more LLM calls.
- A teleportation variant shows it isn't just relying on memorized Sokoban solutions from the internet.
- Optimism doesn't help much here because the reward is dense and random exploration finds it quickly.

### Minigrid

- The environments test different kinds of transfer: empty, door/key, unlock, fetch, and unlock-pickup.
- Transfer speeds up learning.
- Optimism helps more here and lets WorldCoder solve the harder environments.
- Without optimism, it struggles to follow new instructions, so optimism also helps catch LLM mistakes.
- PPO needs 10⁴–10⁵ steps on easy levels and fails UnlockPickup even after 3×10⁸ steps.

### AlfWorld

- A household robot planning domain.
- Uses MCTS with a BM25 text-similarity heuristic to handle sparse reward and long horizons.
- Solves the task in the first episode after around 20 exploratory steps, then makes small corrections in later episodes.
- Optimism is necessary for any non-zero performance.

## Conclusion

WorldCoder shows that an LLM works best as a programmer of world models rather than as a planner or simulator. By writing its knowledge as code, the agent learns from far fewer interactions than deep RL, avoids the per-action LLM cost of ReAct-style agents, and ends up with a model that humans can read and edit. The optimism constraint (φ₂) is what makes exploration goal-directed, and it matters most in sparse-reward settings like Minigrid and AlfWorld. The main limitations are that it assumes deterministic dynamics and needs symbolic, discrete states, so extending it to noisy, perception-heavy environments is the natural next step.
                
---

## Applications and Limitations {#applications-and-limitations}

Given the capabilities of ProgPrompt<d-cite key="singh_progprompt_2023"></d-cite> and WorldCoder<d-cite key="tang_worldcoder_2024"></d-cite> what are the current applications and limitations of these methods?

### Applications {#applications}

### Limitations {#limitations}

---

## Questions and Answers {#questions-and-answers}

This section provides common questions related to the presented topics. *If questions ask to take a side, we provide answers to support both sides.*

### ProgPrompt Questions {#progpromt-questions}

Questions and answers related to ProgPrompt<d-cite key="singh_progprompt_2023"></d-cite>.

### WorldCoder Questions {#worldcoder-questions}

Questions and answers related to WorldCoder<d-cite key="tang_worldcoder_2024"></d-cite>.
