---
layout: distill
title: "Feedback and Correction"
description: "Interactive policy learning through human interventions, corrective teleoperation, and natural language corrections."
date: 2026-10-14
future: true
htmlwidgets: true
ready: false

# Reporter team authors
authors:
  - name: "Jinyoon Kim"
  - name: "Austin Song"
  - name: "Boyang Yan"
  - name: "Sai Tarra"
  - name: "Harris Ilyas"

bibliography: 2026-10-14-feedback-and-correction.bib

toc:
  - name: "Introduction"
  - name: "Yell At Your Robot: Improving On-the-Fly from Language Corrections"
    subsections:
      - name: "Yell At Your Robot: Motivation and Problem"
      - name: "Yell At Your Robot: Method and Formulation"
      - name: "Yell At Your Robot: Experiments and Results"
      - name: "Yell At Your Robot: Limitations and Takeaways"
  - name: "Learning to Learn Faster from Human Feedback with Language Model Predictive Control"
    subsections:
      - name: "Language Model Predictive Control: Motivation and Problem"
      - name: "Language Model Predictive Control: Method and Formulation"
      - name: "Language Model Predictive Control: Experiments and Results"
      - name: "Language Model Predictive Control: Limitations and Takeaways"
  - name: "Comparison and Synthesis"
  - name: "Classroom Questions and Discussion"
    subsections:
      - name: "YAY Robot Questions"
      - name: "Language Model Predictive Control Questions"
      - name: "General Discussion"
---

## Introduction

## Yell At Your Robot: Improving On-the-Fly from Language Corrections

### Yell At Your Robot: Motivation and Problem

A robot may know how to grasp an object and how to place it into a bag, yet still struggle to pack several objects in sequence. The bag can deform, an object can catch on its opening, or a grasp can leave the object in an awkward orientation. Completing the task requires recognizing when a step has gone wrong and choosing an adjustment before continuing. As a task gets longer, there are more opportunities for these execution errors to interrupt the plan.

A hierarchical controller separates this problem into two decisions: a **high-level policy** chooses the instruction to follow, and a **low-level policy** carries out the corresponding movement. However, the high-level policy needs experience with what this particular robot can successfully execute, including how its skills fail. Collecting complete demonstrations of long tasks and their recoveries is expensive. A planner with broad language knowledge may also propose a reasonable instruction without recognizing that the robot needs a small physical adjustment first. <d-cite key="shi2024yell"></d-cite>

Yell At Your Robot (YAY Robot) uses **human language corrections as supervision for the high-level policy**. A person observing an unsuccessful movement can provide a corrective instruction, allowing the robot to change its behavior during that attempt. The system then records the correction so that the high-level policy can learn to select it in similar situations later. This gives language feedback two roles: guiding execution now and improving future autonomous instruction selection. The approach assumes that the low-level controller already knows a sufficiently broad set of task and correction skills. <d-cite key="shi2024yell"></d-cite>

### Yell At Your Robot: Method and Formulation

{% include figure.liquid
   path="assets/img/2026-10-14-feedback-and-correction/yay-robot-figure-3-method-overview.png"
   class="img-fluid rounded z-depth-1"
   alt="YAY Robot overview showing visual observations, a high-level language policy, a frozen low-level controller, and human verbal intervention."
   caption="Method overview, reproduced from Shi et al. (2024), Figure 3."
   zoomable=true
   avoid_scaling=true
%}

The overview shows how observations, instructions, and actions connect. Camera images inform the high-level policy, which selects a language command. The low-level policy combines that command with its observations to produce joint targets. The human intervention joins the language pathway, so a correction changes the instruction passed to the controller. The orange arrow shows how recorded language feedback later updates the high-level policy; the snowflake marks the low-level policy as frozen during this phase. These two pathways correspond to the intervention rule and post-training update explained below. <d-cite key="shi2024yell"></d-cite>

#### Two Policies Connected by Language

Let $$o_t$$ denote the robot's observation at time $$t$$, $$a_t$$ its motor action, and $$l_t$$ a language instruction. Observations include camera images and robot joint information; the actions are target joint positions. The low-level policy $$\pi_L(a_t \mid o_t,l_t)$$ executes the instruction. The high-level policy uses a recent observation history, written $$o_{t-k:t}$$, to select that instruction. History helps distinguish situations that look similar in a single frame but occur at different stages of a task.

The initial dataset $$\mathcal D$$ contains robot demonstrations annotated with language. The operator narrates a skill while teleoperating, and the audio is transcribed and aligned with the trajectory. The dataset includes correction and recovery skills as well as the main task steps. The authors also mark corrections during collection so that preceding erroneous segments can be filtered out of training. <d-cite key="shi2024yell"></d-cite>

#### Learning to Execute Instructions

The low-level policy is initially trained through **behavior cloning**: predict the action that the demonstrator took under the same observation and instruction. Writing the predicted action as $$\hat a_t=f_{\theta_L}(o_t,l_t)$$, the basic objective is

$$
\large
\min_{\theta_L}
\mathbb E_{(o_t,l_t,a_t)\sim\mathcal D}
\left[\mathcal L_{\mathrm{BC}}\left(f_{\theta_L}(o_t,l_t),a_t\right)\right].
$$

This is the action-imitation objective in Equation (1) of the paper, expressed using an explicit prediction function. The loss measures how far the predicted movement is from the demonstrated movement.

In the implementation, the authors adapt **Action Chunking with Transformers (ACT)** to predict a sequence of actions rather than one action at a time, using an $$\ell_1$$ imitation loss. EfficientNet encodes camera images, DistilBERT encodes the instruction, and FiLM layers condition the visual features on language before the ACT controller predicts joint targets. Language therefore influences how the low-level skill is executed. <d-cite key="shi2024yell"></d-cite>

{% include figure.liquid
   path="assets/img/2026-10-14-feedback-and-correction/yay-robot-figure-4-policy-architecture.png"
   class="img-fluid rounded z-depth-1"
   alt="Policy architecture with observation-history encoding and instruction retrieval on the left, and a language-conditioned ACT controller predicting joint targets on the right."
   caption="Policy architecture, reproduced from Shi et al. (2024), Figure 4."
   zoomable=true
   avoid_scaling=true
%}

Read the architecture from left to right. In the left panel, camera-image histories pass through the image encoder and Transformer to predict a language embedding. An embedding represents an instruction as a vector, allowing the prediction to be compared with the available commands. The nearest command becomes the language input to the right panel. There, DistilBERT encodes the instruction, and FiLM uses it to modulate the visual features, helping the controller interpret the scene in relation to the requested behavior. The ACT Transformer combines these features with joint information to predict a sequence of joint targets. The next subsection explains how the instruction-retrieval step is trained. <d-cite key="shi2024yell"></d-cite>

#### Selecting an Instruction from Recent Observations

The high-level policy predicts a **language embedding** from recent visual observations. A frozen CLIP-initialized Vision Transformer encodes the images, and additional trainable layers produce the predicted embedding. The implementation uses up to four observations spaced one second apart.

Let $$h_t=f_{\theta_H}(o_{t-k:t})$$ be the predicted embedding and $$e_j$$ the DistilBERT embedding of candidate instruction $$l_j$$. The model scores each candidate with cosine similarity and applies a softmax:

$$
\large
z_j=\frac{h_t^\top e_j}{\|h_t\|\,\|e_j\|},
\qquad
P_{\theta_H}(l_j\mid o_{t-k:t})
=\frac{\exp(z_j/\tau)}{\sum_m\exp(z_m/\tau)}.
$$

The learned temperature $$\tau>0$$ controls how concentrated the probabilities are: a smaller temperature favors the highest-scoring instruction more strongly. During inference, the policy selects the command with the highest similarity from the commands represented in the dataset. The selected command is then passed to the low-level policy.

The high-level training objective is cross-entropy against the demonstrated instruction:

$$
\large
\min_{\theta_H}
\mathbb E_{(o_{t-k:t},l_t)\sim\mathcal D}
\left[-\log P_{\theta_H}(l_t\mid o_{t-k:t})\right].
$$

This expands the cross-entropy objective in Equation (2): increase the probability of the instruction associated with that observation history. <d-cite key="shi2024yell"></d-cite>

#### Following a Human Correction During Execution

During autonomous operation, the high-level policy selects an instruction $$l^H$$. A human can interrupt and supply a corrective instruction $$l^{\mathrm{user}}$$, which temporarily overrides that selection. Equation (3) describes the switch:

$$
\large
\pi_{\mathrm{deploy}}(a_t\mid o_{t-k:t})=
\begin{cases}
\pi_L(a_t\mid o_t,l^{\mathrm{user}}), & \text{if the human intervenes},\\
\pi_L(a_t\mid o_t,l^H), & \text{otherwise}.
\end{cases}
$$

For example, if the robot needs a directional adjustment while inserting an object into a bag, the human can give that correction instead of letting it continue the current instruction. The same low-level controller executes the new command. **Responding to a correction does not itself update the model's weights**; it changes the instruction that guides the current behavior. The system saves the observation history and human instruction as a training example. <d-cite key="shi2024yell"></d-cite>

#### Learning to Select Corrections in Future Attempts

After collecting feedback, the system fine-tunes the high-level policy using the original data and the correction data, with the same cross-entropy objective. Equation (4) summarizes successive rounds:

$$
\large
\pi_H^{(n+1)}=
\mathrm{PostTraining}\left(
\pi_H^{(n)},
\mathcal D\cup\bigcup_{i=1}^{n}\mathcal D_{\mathrm{corr}}^{(i)}
\right).
$$

Here, $$\mathcal D_{\mathrm{corr}}^{(i)}$$ contains the corrections collected in round $$i$$. These examples expose the high-level policy to situations encountered during actual robot execution, including failures that need corrective instructions. <d-cite key="shi2024yell"></d-cite>

**The low-level controller remains frozen throughout post-training.** A correction example therefore needs the observations and the human instruction, without an additional teleoperated motor-action demonstration. The system learns when to invoke existing skills and corrections; learning a new physical skill would require improving the low-level controller as well. This distinction explains how verbal feedback can improve autonomous behavior while keeping the underlying motor skills fixed. <d-cite key="shi2024yell"></d-cite>

### Yell At Your Robot: Experiments and Results

### Yell At Your Robot: Limitations and Takeaways

## Learning to Learn Faster from Human Feedback with Language Model Predictive Control

### Language Model Predictive Control: Motivation and Problem

### Language Model Predictive Control: Method and Formulation

### Language Model Predictive Control: Experiments and Results

### Language Model Predictive Control: Limitations and Takeaways

## Comparison and Synthesis

## Classroom Questions and Discussion

### YAY Robot Questions

1. **Q: Is YAY Robot learning a general recovery strategy, or simply imitating the corrections provided by humans?**

   **A (class discussion):** There was no definitive way identified in the discussion to distinguish between these possibilities. Zero-shot evaluation on previously unseen failure situations was suggested as a way to investigate this question. If the robot can recover in those situations without additional human corrections, that would provide evidence that its learned behavior generalizes beyond the corrections seen during training.

<!-- Add further YAY Robot questions as numbered Q&A entries above. -->

### Language Model Predictive Control Questions

<!-- Add questions and answers from the class discussion here. -->

### General Discussion

1. **Q: What additional forms of human input could complement language instructions to help the robot understand more complex corrections?**

   **Discussion:** One suggestion from the class was to combine language instructions with human gestures captured through vision. For example, pointing could help indicate the direction in which the robot should move or identify the object a person is referring to. Combining verbal instructions with these visual cues could make directions clearer and provide a richer interface for more complex human instructions.

<!-- Add further general discussion questions as numbered entries above. -->
