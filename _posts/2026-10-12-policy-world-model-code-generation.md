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

---

## Applications and Limitations {#applications-and-limitations}

Given the capabilities of ProgPrompt<d-cite key="singh_progprompt_2023"></d-cite> and WorldCoder<d-cite key="tang_worldcoder_2024"></d-cite> what are the current applications and limitations of these methods?

### Applications {#applications}
Both ProgPrompt and WorldCoder exploit the coding capabilities of LLMs to gain access to its vast knowledge while also restricting the possibility of hallucinations. Forcing the model to write in code provides a structured syntax with automatic validation, as code is easier to validate than natural language. Thus, these methods would be especially beneficial for any kind of tasks that involve non-conventional robots or unfamiliar environments.

LLMs are exceptionally good at generalization, recognizing and applying learned patterns to unseen tasks. However, this is also harmful if the LLM is used for more specialized tasks. In robotics, an LLM, without any constraints, may assume that the robot is a humanoid and give commands based on this assumption. But, if the robot is a robotic dog or even an aquatic robot, then the commands the model gives is not applicable. On the other hand, using the LLM via a code interface enables the model to only access commands the robot is able to execute, which gives ProgPrompt and WorldCoder the advantage over vanilla LLMs. These approaches can better manipulate the robot to complete the task. 

Another application is for unfamiliar tasks or environments (i.e., tasks or environments extremely different from those the LLM was trained on). In this scenario, the model's knowledge provides little benefit other than commonsense knowledge, meaning that it is possible that using the LLM for planning is closer to random. However, WorldCoder provides a simulated environment for the robot to learn a suitable policy, and ProgPrompt provides the executable actions the model can use to explore the environment and complete the task. For both approaches, having a code interface enables better adaptability of the robots to unseen environments. 

### Limitations {#limitations}
Although code provides more structured syntax to restrict model output and prevent hallucinations, there are some key limitations of this type of approach.
1. _Executability does not equate to better performance._ These approaches only provide an alternative communication interface (via code), but it does not necessarily translate to improved task performance. We can even argue that using these approaches increases the model capability requirements, as the model not only needs to understand code but also generate valid code that accomplishes the goal. Smaller language models might be suitable for natural language communication but fail to output valid code syntax. 
2. _The possibility of hallucination is reduced, not eliminated._ Even if we restrict model output to only generate code, the model can still generate incomplete or incorrect code. The main benefit is that we can quickly filter out invalid code, such as using a compiler. However, there is also the chance that the model generates syntactically valid but semantically incorrect code, such as calling the wrong function or using the wrong variable.
3. _The code interface requires more human setup._ Both methods require a code API to communicate with the robot, which could restrict the applicability of these methods. Furthermore, prior to using LLMs, a human annotator would need to design a prompt that enables the LLM to understand the code API and use it to generate commands. This type of approach requires a lot of effort to ensure compatibility between the robot and the LLM. 

---

## Questions and Answers {#questions-and-answers}

This section provides common questions related to the presented topics. *If questions ask to take a side, we provide answers to support both sides.*

### ProgPrompt Questions {#progpromt-questions}

Questions and answers related to ProgPrompt<d-cite key="singh_progprompt_2023"></d-cite>.

### WorldCoder Questions {#worldcoder-questions}

Questions and answers related to WorldCoder<d-cite key="tang_worldcoder_2024"></d-cite>.
