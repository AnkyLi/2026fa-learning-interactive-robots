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

Large language models (LLMs) are pre-trained on most of the internet data to form a strong baseline for applying its generality to various different problems. This allows LLMs to applied to robotics scenarios<d-cite key="zeng_large_2025"></d-cite> ranging from high level decision making to lower level controls. LLMs have strong potential to increase the reasoning capabilities of robots by having an LLM connect all of the observations together and produce a reasonable action<d-cite key="wang_actionreasoning_2026"></d-cite> for complex scenarios that are difficult to fully model.

However, there two major challenges for deploying LLMs into physical embodiments:

- LLMs are not pre-trained on robotic embodiments, how can an LLM reasonable know how to utilize the physical space and actions provided by the robot?
- LLMs have the ability to hallucinate, how can the risk of errors due to hallucinations be reduced and improve robustness?

This lecture explores two different approaches to address the challenges:

- **PROGPROMPT: Generating Situated Robot Task Plans using Large Language Models <d-cite key="singh_progprompt_2023"></d-cite>:** Instead of allowing an LLM to interact directly with low level controllers, the LLM is instructed to write a function based, API style code to control the robot. By allowing the LLM to stay in its primary domain, the results should be more consistent since the robot model has already been abstracted into a code object. Through guided examples and self verification, PROGPROMPT is able to complete tasks at a higher rate compared to vanilla LLM inference.
- **WorldCoder, a Model-Based LLM Agent: Building World Models by Writing Code and Interacting with the Environment <d-cite key="tang_worldcoder_2024"></d-cite>:** Instead of using LLMs to purely reason about complex scenarios, have the LLM construct a world model of the environment based on state and reward feedback. This allows the LLM to be able the reference the current world model instead of trying to memorize the states from the context. The constraints increase robustness since the LLM can query the world model to predict the transition function of the environment. The LLM then creates and optimizes a policy to achieve the task based on the world model. Both the policy and world model are updated when key metrics do not align with true observations from the environment.

---

## ProgPrompt {#progprompt}

---

## WorldCoder {#worldcoder}

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
