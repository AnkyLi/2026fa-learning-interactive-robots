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
      - name: "Generic Questions"
---

## Introduction {#introduction}

Large language models (LLMs) are pre-trained on most of the internet data to form a strong baseline for applying its generality to various different problems. This allows LLMs to applied to robotics scenarios<d-cite key="zeng_large_2025"/> ranging from high level decision making to lower level controls. LLMs have strong potential to increase the reasoning capabilities of robots by having an LLM connect all of the observations together and produce a reasonable action<d-cite key="wang_actionreasoning_2026"/> for complex scenarios that are difficult to fully model.

However, there two major challenges for deploying LLMs into physical embodiments:

- LLMs are not pre-trained on robotic embodiments, how can an LLM reasonable know how to utilize the physical space and actions provided by the robot?
- LLMs have the ability to hallucinate, how can the risk of errors due to hallucinations be reduced and improve robustness?

This lecture explores two different approaches to address the challenges:

- **PROGPROMPT: Generating Situated Robot Task Plans using Large Language Models <d-cite key="singh_progprompt_2023"/>:** Instead of allowing an LLM to interact directly with low level controllers, the LLM is instructed to write a function based, API style code to control the robot. By allowing the LLM to stay in its primary domain, the results should be more consistent since the robot model has already been abstracted into a code object. Through guided examples and self verification, PROGPROMPT is able to complete tasks at a higher rate compared to vanilla LLM inference.
- **WorldCoder, a Model-Based LLM Agent: Building World Models by Writing Code and Interacting with the Environment <d-cite key="tang_worldcoder_2024"/>:** Instead of using LLMs to purely reason about complex scenarios, have the LLM construct a world model of the environment based on state and reward feedback. This allows the LLM to be able the reference the current world model instead of trying to memorize the states from the context. The constraints increase robustness since the LLM can query the world model to predict the transition function of the environment. The LLM then creates and optimizes a policy to achieve the task based on the world model. Both the policy and world model are updated when key metrics do not align with true observations from the environment.

---

## ProgPrompt {#progprompt}

---

## WorldCoder {#worldcoder}

---

## Applications and Limitations {#applications-and-limitations}

Given the capabilities of ProgPrompt<d-cite key="singh_progprompt_2023"/> and WorldCoder<d-cite key="tang_worldcoder_2024"/> what are the current applications and limitations of these methods?

### Applications {#applications}

### Limitations {#limitations}

---

## Questions and Answers {#questions-and-answers}

This section provides common questions related to the presented topics. *If questions ask to take a side, we provide answers to support both sides.*

### ProgPrompt Questions {#progpromt-questions}

Questions and answers related to ProgPrompt<d-cite key="singh_progprompt_2023"/>.

#### Q: Is code the right interface for robots, or just the one LLMs happen to be good at?

*Code is the right interface for robots:* Code is an excellent medium for LLMs to interact with robotic systems because, unlike generating raw plans or actions, the code can be analyzed and verified. Even though there is still no explainability for the LLM inference, the chain of thought can be roughly seen through the comments and code structure. The other benefit of code is its reusability because the LLM does not need to generate a primitive action for every step, the function just needs to be generated once and can be queried again. This allows the motion to be more predictable between steps and verifiable by humans offline.

*Code is not an appropriate interface:* By having the LLM write code for interacting with the robot, we rely on the LLM to not hallucinate feasible and safe actions that are actually not achievable or dangerous. A much stronger method would be to combine LLMs with a library of actions using dynamic motion primitives (DMP)<d-cite key="anarossi_keymps_2025"/>. Leveraging DMPs allows the LLM to stay physically grounded with the robot and environment by focusing on utilizing a set of motions to complete a high level task.

#### Q: ProgPrompt hit 90% executability but only 40% success with Codex. Is "executability" really a meaningful metric?

*Executability defines a useful intermediate state:* Executability is the first step in determining whether a task can be achieved because if the LLM creates a broken function then there is zero chance of the task being completed. This metric allows us to approximately determine how many high level logic mistakes were made based on the difference between executability and success rate ($90-40=50\%$). The metric also allows for targeted fixes either structurally, prompt design, examples, etc. since the system is getting stuck without making any progress.

*Executability does not capture anything meaningful:* The authors do not formally define executability which leaves the readers guessing what it actually represents. Assuming it means that the state checks through assertions are violated or the code is un-runnable, then the metric does not have a strong use in the analysis. It is is too broad of a metric to allow us to determine what went wrong during the process. Without more details we do not know why the LLM failed to generate the code. Maybe the LLM hallucinated an feasible action, the current state, objects in the scene, etc. but we do not know without finer metrics.

### WorldCoder Questions {#worldcoder-questions}

Questions and answers related to WorldCoder<d-cite key="tang_worldcoder_2024"/>.

#### Q: WorldCoder's "optimism" makes the agent believe the goal is reachable and test it. But is that safe to deploy on a real robot?

*Optimism is risky and unsafe for real deployments:* Being optimistic can cause the robot to move towards unsafe regions and attempt tasks that a reasonable policy would not propose. Even though safety filters such as control barrier functions<d-cite key="ames_control_2017"/> can be employed to guarantee safety, we should be designing our systems to be safe at every layer and not rely on a catch all at the end. So the policy should be grounded with a notion of safety and risk to prevent overly optimistic goals from pushing robot into situations that it should avoid. Maybe this is implemented as a direct foil to optimism with something like pessimism, where certain conditions cause the robot to be wary of unsafe scenarios.

*Optimism is conditionally safe for the real world:* Without constraints, overly optimistic goals can be risky but since WorldCoder can analyze rollouts based on the current world model then it should be able to determine whether a goal is unsafe before becoming unsafe. However, this relies on the world model having sufficient alignment and correlation with the real world. If there are parts of the world that are not modeled due to occlusions or sensing limitations then the policy could direct the robot to an unsafe goal. So given an accurately created world model then the optimism trait should not hinder robot safety.

#### Q: Coding agents already write and test software. Could they eventually build and debug robotic systems autonomously?

*Coding agents are not capable to build robotic systems:* Coding agents can do a lot of things very efficiently such as running test suites for pinning down specific errors and then proposing a patch. This is done iteratively until the task is complete. When the task is just coding this can be achieved but not guaranteed because LLMs have a tendency to scope creep<d-cite key="qu_overeager_2026"/> and might end up changing the problem constraints or objectives during the process. This is already an issue for solving an end to end problem but since robotic systems need physical embodiments, the coding agent needs access to those embodiments. Physical systems are notoriously difficult to work with and most require physically working with them to run experiments. Without testing a system on physical hardware, even in a lab setting, then the efficacy of the system is still unknown and not validated. The best chance a coding agent would have at building and debugging a whole robotic system is to leverage services that abstract the physical robots away from the user. The Robotarium<d-cite key="wilson_robotarium_2020"/> allows users to test code in simulation and then remotely deploy to physical robots in a managed environment. But this is just a setup for niche experiments and does not allow for real world experiments outside of the laboratory.

### Generic Questions {#generic-questions}

Questions and answers related to ProgPrompt<d-cite key="singh_progprompt_2023"/> and WorldCoder<d-cite key="tang_worldcoder_2024"/>.

#### Q: Neither approach updates the LLM itself. Is that a strength (interpretable, editable) or a restriction (ceiling)?

*In context learning is a strength:* Updating LLM weights is computationally costly and risky for shifting the baseline as the task progresses. The only way to reasonably try and update the LLM weights is with fine-tuning because the model is too big to fully train. Since fine-tuning focuses on the layers closest to the output layer, significant changes in outputs can occur from limited training examples. This is risky for model that is supposed to be used in robotic systems where the environment can change quickly. Just because the weights are not updated does not mean the model does not learn and adapt to the prompts it has been given. In context learning<d-cite key="dong_survey_2024"/> allows the model to adjust its inference by holding information in its context window. This is an effective method for allowing the model to "learn" the preferences or requirements for the tasks.

*Not updating the weights is restricting:* Without updating weights the LLM is relying on its context to learn how to respond to a task it has previously encountered. The LLM can not just be blindly finetuned with the new experiences and must go through an algorithm designed to handle streams of new data for fine-tuning<d-cite key="liu_sample_2026"/>. If such a method is implemented and does not take too much compute resources, then it will be beneficial to train the new learned concepts into the model for permanent reuse.
