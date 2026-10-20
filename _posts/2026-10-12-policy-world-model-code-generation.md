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
    subsections:
      - name: "Classical & LLM Planners"
      - name: "The Grounding Gap"
      - name: "Problem Formulation"
      - name: "Method"
      - name: "Evaluation"
        subsections:
          - name: "Simulation"
          - name: "Real-World Experiments"
      - name: "Discussion"
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

**Motivation**: Household tasks require both commonsense world understanding as well as situated knowledge of the current environment. LLMs know a lot about the world, and the question becomes how to leverage this understanding in an embodied robot. 

**ProgPrompt's** key idea is to introduce situated-awareness in LLM-based robot task planning by framing it as Python program generation. The LLM is promped with pythonic code form specifications of actions, objects, and examples, and it completes the program for a new task.

### Classical & LLM Planners

**Classical Task Planning**: Task planning traditionally relies on symbolic search (e.g. PDDL, STRIPS) over predefined domain models, but it struggles to scale in open-world environments.

**LLMs as Planners**: LLMs (e.g. GPT-3, Codex) possess vast commonsense knowledge. Recent work has used natural language (NL) instructions for zero-shot and few-shot task decomposition.

### The Grounding Gap

> **LLMs generate plausible sounding text, but they lack physical embodiment and environmental awareness**

**API Mismatch**: The LLM proposes actions that the robot cannot physically perform.

**Object Hallucination**: The LLM referencse objects not in the current room.

**No State Feedback**: The open-loop plan generation cannot adapt when a precondition fails.

### Problem Formulation

The task planning is modeled as a tuple:

$$
\langle O, P, A, T, I, G, t \rangle
$$

- $$O$$: set of all objects available in the environment
- $$P$$: set of properties of the objects; informs object affordances
- $$A$$: set of executable actions, which depends on the current state $$s \in S$$
- $$T$$: transition model $$T: S \times A \rightarrow S$$
- $$I, G$$: initial and goal states
- $$t$$: high-level task description 


### Method

{% include figure.liquid path="assets/img/2026-10-12-policy-world-model-code-generation/r2-p1-arch.png"
class="img-fluid rounded z-depth-1"
caption="Overview of ProgPrompt architecture highlighting the components of the prompting, generated function, state feedback, and execution."
%}

ProgPrompt utilizes a *PROMPT for Planning* and a *PROMPT for Feedback* to obtain and refine a generated plan from the LLM. Key componenets of this process are the provided environmental information and examples, prompt comments, and assertions/assertion checks, discussed in deatil below.

**Robot Plan Representation**

Robot plans are represented as Pythonic programs. The prompt is structured as Pythonic code, and the LLM completes it. The plan function consists of API calls to action primitives that take objects as arguments, summary comments, and assertions to track execution. Rather than scoring candidates or generating text, the LLM generates an entire executable plan program.

**Why Code?** The motivation is that LLMs are heavily pretrained on code, including many programming tutorials and lots of documentation on the web. Code carries strict syntax, functional constraints, and procedural logic that natural language lacks. NL can often be ambiguous, whereas the function call either does or does not match the API.

**Prompt Design**

The prompt is written as a Python file with 4 parts:
1. **Action Primitives as Imports**
2. **Available Objects List**
3. **Example Tasks**
4. **Open-Ended Function Header**

**Task Plan Generation & Execution**

{% include figure.liquid path="assets/img/2026-10-12-policy-world-model-code-generation/r2-p1-funcex.png"
class="img-fluid rounded z-depth-1"
caption="Pythonic ProgPrompt plan for "put salmon in the microwave"."
%}

Generated plan functions, as shown in the figure, contain the following key components:
- **API calls** to action primitives with objects as arguments
- **NL comments** to break down tasks, focus on immediate goals, and reduce repetitive output. This process encourages and mimics "chain of thought" style reasoning and prompting.
- **Assertions with recovery actions**, which check preconditions before acting and fix the state when a check fails.

### Evaluation

The authors evaluated ProgPrompt across virtual simulation and real-world experiment domains.

**Metrics**

- Success Rate (SR): the fraction of task executions that successfully achieved all task relevant goal conditions
- Executability (Exec): the fraction of actions generated in the plan that are executable by the agent in that environment
- Goal Conditions Recall (GCR): compares the final state achieved against the ground truth goal conditions

#### Simulation

**Simulation Setup**: The simulation was run in VirtualHone, a deterministic simulation platform for typical household activities.
<!-- , with the action space $$A=\{\text{grab, putin, putback, walk, find, open, close, switchon, switchoff, sit, standup}\}$$. They experiment with 3 environments, each with 115 unique object instances, and create a dataset of 70 tasks. Each result is averaged over 5 runs across 10 tasks. -->

**Simulation Process**:
1. Simulator extracts the scene state.
2. ProgPrompt builds the LLM Prompt.
3. The LLM process the prompt and outputs the completed function.
4. The interpreter processes the code line-by-line.
5. The Simulator executes via API calls.
6. Repeat until the program is finished.

{% include figure.liquid path="assets/img/2026-10-12-policy-world-model-code-generation/r2-p1-simtable-comparison.png"
class="img-fluid rounded z-depth-1"
%}

- ProgPrompt w/ GPT-3 outperformed the zero-shot planner baseline and LangPrompt on every metric.
- Codex achieved the highest overall performance since it was explicitly trained on programming language code.
- Davinci underperformed relative to GPT-3 likely due to prompt length constraints limiting task example quantity.
- Removing comments reduced performace across all metrics
- Including assertion checks and recovery steps improved performance.

{% include figure.liquid path="assets/img/2026-10-12-policy-world-model-code-generation/r2-p1-simtable-tasks.png"
class="img-fluid rounded z-depth-1"
%}

- Tasks similar to prompt examples have higher GCR.
- No longer task with 11-18 ground truth actions were fully completed.
- High executability does not guarantee high GCR, since some tasks have multiple appropriate foal states.

{% include figure.liquid path="assets/img/2026-10-12-policy-world-model-code-generation/r2-p1-simtable-scenes.png"
class="img-fluid rounded z-depth-1"
%}

- ProgPrompt transfers to new scenes with the object list appended after example tasks in the prompt.
- ProgPrompt can infer that cutlery refers to spoons and forks even when only used to refer to knives in examples.

#### Real-World Experiments

**Experiment Setup**: A Franka-Emika Panda robot with a parallel jaw gripper was used to perform pick-and-place operations.

**Experiment Process**:
1. VLID detects items in the environment.
2. ProgPrompt builds the LLM Prompt.
3. The LLM process the prompt and outputs the completed function.
4. The interpreter processes the code line-by-line and maps to the environment.
5. A low-level control stack executes the plan.
6. Repeat until the program is finished.

{% include figure.liquid path="assets/img/2026-10-12-policy-world-model-code-generation/r2-p1-exp.png"
class="img-fluid rounded z-depth-1"
caption="Robot plan execution rollout example on a sorting task."
%}

- The LLM recognizes the banana and strawberry as ruits.
- The LLM ignores the distractor object.

{% include figure.liquid path="assets/img/2026-10-12-policy-world-model-code-generation/r2-p1-exptable.png"
class="img-fluid rounded z-depth-1"
%}

- The only failures occured on the sorting task.
- Failure causes included distractors as well as an unexpected gripper failure.


### Discussion

**ProgPrompt**: an LLM prompting scheme for robot task planning that leverages the strength of LLMs in commonsense reasoning and code understanding to prompt for executable plans as programs.

*Recommendation from authors*: utilize a program-like prompt for LLM-based task planning and execution.

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
