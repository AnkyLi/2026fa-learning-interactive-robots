---
layout: distill
title: "Feedback and Correction"
description: "Interactive policy learning through human interventions, corrective teleoperation, and natural language corrections."
date: 2026-10-14
future: true
htmlwidgets: true
ready: true

# Reporter team authors
authors:
  - name: "Jinyoon Kim"
    affiliations:
      - name: "University of Virginia"
  - name: "Austin Song"
    affiliations:
      - name: "University of Virginia"
  - name: "Boyang Yan"
    affiliations:
      - name: "University of Virginia"
  - name: "Sai Tarra"
    affiliations:
      - name: "University of Virginia"
  - name: "Harris Ilyas"
    affiliations:
      - name: "University of Virginia"

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
  - name: "Discussion"
---

*Author affiliation (all authors): University of Virginia.*

## Introduction
Feedback is central to interactive robot learning because a robot's first attempt at a task is often imperfect. Even a capable robot may misinterpret a human instruction, encounter an unexpected object configuration, or execute a motion that requires adjustment. A useful interactive system must therefore do more than follow an initial command: it must interpret corrections, modify its behavior, and ideally become easier to teach through experience.

This lecture examines two approaches to that problem. Yell At Your Robot (YAY Robot) uses human verbal interventions to override high-level skill selection during robot execution, then fine-tunes the high-level policy so that it can independently choose appropriate corrections in the future. Learning to Learn Faster from Human Feedback with Language Model Predictive Control (LMPC) studies a different interaction loop: a human gives natural-language feedback to a code-generating language model, which learns to produce robot programs that require fewer subsequent corrections. Whereas YAY Robot learns which physical skill or recovery instruction to invoke, LMPC learns how to respond to conversational teaching and select an effective next program.

Together, the papers illustrate two complementary meanings of learning from feedback. Within an interaction, human input can immediately change the robot's next behavior. Across interactions, recorded corrections can be used to update a learned model, making future interactions more successful. The following sections explain each method, discuss its empirical evidence and limitations, and consider what it would take to combine adaptive planning with reliable physical recovery.

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

The authors tested YAY Robot on three tasks: putting a Sharpie, tape holder, and sponge into a Ziploc bag; making trail mix; and cleaning gummy bears off a plate with a sponge. These tasks are difficult because small mistakes can build up over several steps. They ran 20 trials for each evaluation and measured success on both individual steps and the full task. <d-cite key="shi2024yell"></d-cite>

They compared three versions: the Base Policy, which worked without extra help; YAY Robot + Oracle Corrections, where a person could give spoken corrections while the robot worked; and YAY Robot After Finetuning, which learned from those corrections and was then tested without further help. The low level controller stayed the same, so the robot learned when to choose instructions it already knew how to follow rather than learning new movements.

The results show that spoken corrections can help the robot succeed, and that some of this improvement carries over to later attempts. Overall, the paper reports about a 20% improvement in success rate over the base policy across the three tasks. Figure 6 compares the results for the base policy, the fine tuned robot, and the robot receiving corrections during the task.

{% include figure.liquid
   path="assets/img/2026-10-14-feedback-and-correction/yay-robot-figure-6-quantitative-evaluations.png"
   class="img-fluid rounded z-depth-1"
   alt="Bar charts comparing the base policy, the fine-tuned YAY Robot, and human-provided corrections across bag packing, trail mix preparation, and plate cleaning."
   caption="Success rates across the three tasks, reproduced from Shi et al. (2024), Figure 6."
   zoomable=true
   avoid_scaling=true
%}

Figure 7 shows that bag packing results improved over repeated rounds of feedback and fine tuning, getting closer to the results achieved when a person corrected the robot directly. The robot also began choosing useful recovery instructions on its own, such as adjusting a gripper before trying again. Figures 9 and 10 show a similar change in plate cleaning: the robot covered more of the plate and used corrective cleaning instructions more often after fine tuning.

{% include figure.liquid
   path="assets/img/2026-10-14-feedback-and-correction/yay-robot-figure-7-iterative-improvement.png"
   class="img-fluid rounded z-depth-1"
   alt="Line chart showing bag-packing success improving over repeated rounds of verbal feedback and fine-tuning, alongside results with human-provided corrections."
   caption="Improvement in bag-packing performance over repeated fine-tuning rounds, reproduced from Shi et al. (2024), Figure 7."
   zoomable=true
   avoid_scaling=true
%}

The authors also compared YAY Robot with other ways of choosing instructions. Its learned approach generally did better than a flat imitation learning policy, especially during later steps of a task. A fixed sequence of instructions could not respond to unexpected mistakes, and GPT-4V sometimes misunderstood where objects were or what the robot was doing. These results suggest that the robot benefits from a learned policy that can choose instructions based on the situation.

{% include figure.liquid
   path="assets/img/2026-10-14-feedback-and-correction/yay-robot-figure-8-ablation-high-level-policy.png"
   class="img-fluid rounded z-depth-1"
   alt="Bar charts comparing YAY Robot's learned high-level policy with a fixed instruction sequence, GPT-4V, and one-hot skill labels."
   caption="Comparison of alternative ways to select high-level instructions, reproduced from Shi et al. (2024), Figure 8."
   zoomable=true
   avoid_scaling=true
%}

{% include figure.liquid
   path="assets/img/2026-10-14-feedback-and-correction/yay-robot-figure-9-policy-proficiency-finetuning.png"
   class="img-fluid rounded z-depth-1"
   alt="Plate-cleaning heatmaps comparing cleaning coverage before and after fine-tuning; brighter areas show where wiping was more effective."
   caption="Plate-cleaning coverage before and after fine-tuning, reproduced from Shi et al. (2024), Figure 9."
   zoomable=true
   avoid_scaling=true
%}

{% include figure.liquid
   path="assets/img/2026-10-14-feedback-and-correction/yay-robot-figure-10-correction-skill-ratio.png"
   class="img-fluid rounded z-depth-1"
   alt="Charts showing that the robot used corrective cleaning instructions more often after fine-tuning."
   caption="Changes in the use of corrective cleaning instructions after fine-tuning, reproduced from Shi et al. (2024), Figure 10."
   zoomable=true
   avoid_scaling=true
%}

### Yell At Your Robot: Limitations and Takeaways

YAY Robot can only follow a correction if its low level controller already knows how to carry out that instruction. Fine tuning helps it choose better instructions, but does not teach it entirely new movements. The results suggest that improving the robot's existing skills is still important. <d-cite key="shi2024yell"></d-cite>

The system also relies on spoken instructions and cannot easily understand pointing or gestures. The authors suggest that future systems could combine speech with these other ways of communicating. Their GPT-4V results also show that a general vision language model may misunderstand object positions or what is happening during a task.

The quality of the training data matters, too. Adding more examples did not always improve results when the extra data included poor or unsuccessful behavior. Overall, YAY Robot shows that simple spoken corrections can help a robot recover in the moment and make better choices next time. However, its performance still depends on the skills it already has, the quality of its training data, and people's ability to explain corrections in words.


## Learning to Learn Faster from Human Feedback with Language Model Predictive Control

### Language Model Predictive Control: Motivation and Problem
A general-purpose robot may be controlled by a large language model (LLM) that translates an instruction such as "make the robot crouch" or "move the can into the drawer" into executable code. Non-expert users can then inspect the behavior and issue follow-up corrections. This kind of **in-context learning** is useful because it requires no weight update during a conversation; the model can react using the current dialogue history. However, adaptation based on the prompt is temporary, may disappear when earlier messages exceed the context window, and does not necessarily make the model better at responding to the next user's corrections.

The paper asks how a code-writing LLM can become more **teachable**: how successfully and efficiently can a non-expert bring it to a desired robot behavior through a limited number of conversational corrections? This is distinct from measuring whether the first code sample works. An initially incorrect output may still be useful if the model understands feedback and quickly produces a successful revision. Conversely, a system that repeatedly makes similar mistakes imposes a large teaching burden even if it occasionally succeeds.

The authors distinguish two adaptation timescales. **Fast adaptation** happens online as the LLM conditions on the current chat and generates revised code. **Slow adaptation** happens offline, when interactions from earlier teaching sessions are used to fine-tune the LLM. The proposed Language Model Predictive Control (LMPC) connects these two: it uses previously observed teaching trajectories to learn about future dialogue and searches for next responses likely to reach task success with fewer corrections. Unlike conventional MPC, which normally predicts physical state evolution under control inputs, LMPC applies predictive control to the *human–LLM interaction sequence*. <d-cite key="liang2024learning"></d-cite>

### Language Model Predictive Control: Method and Formulation

#### Human–Robot Dialogue as a Sequential Decision Problem

A teaching session begins with a system prompt $$P$$ describing the robot's capabilities and code interface. At dialogue turn $$t$$, the human provides language $$h_t$$, and the LLM returns executable robot code $$c_t$$. The code is evaluated through execution or simulation, after which the human may give another instruction or declare success. A session can therefore be represented schematically as

$$
\tau = (P,h_0,c_0,h_1,c_1,\ldots,h_T,c_T,r),
$$

where $$r$$ indicates the final task outcome. In the authors' partially observable Markov decision process (POMDP) interpretation, **human language is an observation** and **robot code is an action**. The hidden state includes aspects of the user's intended behavior and the execution outcome not fully described by the messages. The model observes conversation history rather than the entire physical state. <d-cite key="liang2024learning"></d-cite>

The practical objective is to choose the next code response that leads to a successful session in as few additional interactions as possible. A convenient conceptual expression of this goal (not a verbatim equation from the paper) is

$$
 c_t^* \approx \underset{c_t}{\arg\max}\;\Pr\bigl(\text{task success within the remaining feedback budget}\mid P,h_{0:t},c_{0:t-1},c_t\bigr).
$$

The key distinction is that a good next code response need not merely resemble the next response in an existing dialogue; it should improve the likely *future teaching outcome*.

#### LMPC-Rollouts: Predicting Future Teaching Interactions

The teaching interface shown in lecture slide 20 has the LLM generate **reward code**, which is passed to the robot controller to produce behavior that the user then evaluates. Thus, the code in the dialogue is not simply a low-level motor action; it specifies a programmatic target for the controller.

The **LMPC-Rollouts** approach fine-tunes an LLM on collected human–robot interactions so it can continue a partially observed conversation, including likely future human feedback and subsequent code responses. This gives the LLM the role of a learned interaction-dynamics model. At inference time, the controller considers possible code responses, generates imagined future dialogue continuations, and uses their predicted outcomes to choose a promising next response. <d-cite key="liang2024learning"></d-cite>

The lecture presentation summarizes the inference rule particularly concretely: sample hypothetical conversations from the current history, select the **shortest completed rollout** indicating predicted success, return its **first code response**, and then repeat the process after receiving *actual* feedback. This is a heuristic based on the model's simulated completions, not a guarantee of shortest real-world teaching time.

The process follows the receding-horizon principle of **model predictive control**:

1. Observe the actual human instruction and the current chat history.
2. Consider candidate next robot-code outputs and predict potential follow-up feedback and revisions.
3. Prefer a candidate whose predicted continuation reaches success with fewer additional interactions.
4. Execute or present only the **next** selected code response, obtain real human feedback, and replan using the updated history.

For example, a user may ask a robot dog to adopt a pose. A proposed program might place its body too high. A rollout can represent a possible correction such as "lower the body" and the resulting code revision. By evaluating candidate continuations, the model seeks an initial or revised program that makes reaching the desired pose easier. These predictions are **hypotheses about a future interaction**, not guarantees that a human will provide the predicted feedback.

#### LMPC-Skip: Learning to Jump Directly to Successful Code

**LMPC-Skip** offers a more direct alternative: rather than simulate a sequence of future corrections during selection, the model is trained to predict a later successful code response from an earlier dialogue prefix. In effect, it learns to skip intermediate revisions in successful teaching trajectories. This can strengthen the first response because it directly imitates the successful endpoint, but it models the intervening feedback process less explicitly.

The distinction explains an important experimental tradeoff: **Skip** can be strong at producing a useful first program, while **Rollouts** better targets adaptation over multiple correction turns. Neither method eliminates the need for real human feedback; both use previous teaching examples to improve future responses. <d-cite key="liang2024learning"></d-cite>

#### Learning from Successful Users and Measuring Teachability

Teaching trajectories differ greatly in quality. Some users communicate corrections efficiently, while others require more iterations or provide ambiguous instructions. The paper explores **top-user conditioning**, which allows the model to learn from a broad range of users while favoring interaction patterns associated with more successful teaching. The experiments test the value of this conditioning through ablations, rather than assuming that more data or only expert-like demonstrations will always be better.

The core metric is **teachability**: both whether a task can eventually be completed and how much conversational feedback is required. This motivates reporting task success across different limits on the number of chat turns, rather than treating one-shot accuracy as sufficient. In the paper's reported unseen-task evaluation, feedback-aware training also reduced average human corrections from approximately **2.4 to 1.9**. <d-cite key="liang2024learning"></d-cite>

### Language Model Predictive Control: Experiments and Results
The authors evaluated LMPC on 78 tasks across five robot embodiments in simulation, with additional tests on two real robots. They collected feedback from 35 non-expert users, who could give up to seven rounds of feedback per session. The evaluation measured task success, the number of chat turns needed for success, and how well the models responded to corrections.

They compared PaLM 2-S, a retrieval-augmented generation (RAG) baseline, LMPC-Skip, and LMPC-Rollouts. Figure 4 plots how success changes as users are allowed more chat turns. On unseen tasks, LMPC-Rollouts reached 66.3% success compared with 39.4% for PaLM 2-S, a 26.9 percentage-point improvement. <d-cite key="liang2024learning"></d-cite>

{% include figure.liquid
path="assets/img/2026-10-14-feedback-and-correction/lmpc-figure-4-teachability.png"
class="img-fluid rounded z-depth-1"
alt="Line charts comparing task success rates as the number of chat turns increases on train and test tasks."
caption="Teachability across training and unseen tasks, reproduced from Liang et al. (2024), Figure 4."
zoomable=true
avoid_scaling=true
%}

Table I shows a tradeoff between the two fine-tuned methods. LMPC-Skip did better on the first instruction (41.7% versus 34.8% on test tasks), but LMPC-Rollouts had more successes after corrective feedback (31.5% versus 17.8%). The authors suggest using Skip initially and Rollouts for later corrections. <d-cite key="liang2024learning"></d-cite>

{% include figure.liquid
path="assets/img/2026-10-14-feedback-and-correction/lmpc-table-1-results.png"
class="img-fluid rounded z-depth-1"
alt="Table comparing overall, one-turn, and multi-turn success rates for PaLM 2-S, RAG, LMPC-Skip and LMPC-Rollouts."
caption="Success rates by method and number of turns, excerpted from Liang et al. (2024), Table I."
zoomable=true
avoid_scaling=true
%}

The authors also tested top-user conditioning. Table II shows that removing it lowered performance, while training only on the top users was also insufficient. This suggests the model benefits from diverse teaching examples while being guided toward more effective feedback styles. <d-cite key="liang2024learning"></d-cite>

{% include figure.liquid
path="assets/img/2026-10-14-feedback-and-correction/lmpc-table-2-top-user-conditioning.png"
class="img-fluid rounded z-depth-1"
alt="Ablation table showing declines in train- and test-task success when top-user conditioning is removed."
caption="Effects of removing top-user conditioning, reproduced from Liang et al. (2024), Table II."
zoomable=true
avoid_scaling=true
%}

To test generalization, the authors trained on Robot Dog, Mobile Manipulator, and ALOHA, then evaluated on two unseen KUKA embodiments. LMPC-Rollouts improved success by 31.5 percentage points over PaLM 2-S on these new robots, even though their APIs and tasks differed from those seen during training. <d-cite key="liang2024learning"></d-cite>

{% include figure.liquid
path="assets/img/2026-10-14-feedback-and-correction/lmpc-table-4-cross-embodiment.png"
class="img-fluid rounded z-depth-1"
alt="Success-rate improvements on training embodiments and previously unseen robot embodiments."
caption="Cross-embodiment generalization, reproduced from Liang et al. (2024), Table IV."
zoomable=true
avoid_scaling=true
%}

Finally, the simulation-trained model was evaluated on real Robot Dog and Mobile Manipulator tasks, shown in Figure 5. Average task success increased from 53.8% to 75%, although the average number of chat turns did not decrease. These results suggest the learned feedback strategy can transfer to real robots, but greater success does not always mean fewer corrections. <d-cite key="liang2024learning"></d-cite>

{% include figure.liquid
path="assets/img/2026-10-14-feedback-and-correction/lmpc-figure-5-real-robot-tasks.png"
class="img-fluid rounded z-depth-1"
alt="Photographs of a real robot dog performing poses and a mobile manipulator opening a drawer and moving a can."
caption="Examples of real-world robot tasks, reproduced from Liang et al. (2024), Figure 5."
zoomable=true
avoid_scaling=true
%}
### Language Model Predictive Control: Limitations and Takeaways
One limitation of LMPC is its high computational cost, since fine-tuning the language model and predicting future conversations require substantial resources, with the robot controller potentially needing around 128 CPU cores.

LMPC also depends on successful teaching interactions for training, which makes it difficult to improve on tasks the base model cannot solve. To add onto this, the system only models language feedback, without incorporating gestures or other forms of human communication.

One important thing to note as well is that repeated fine-tuning did not improve performance either, with a second round decreasing success by 1.9 percentage points on test tasks. Most evaluations were also conducted in simulation, limiting conclusions about real-world generalization. Ultimately, LMPC shows that learning from human feedback improves robot adaptability, but computational cost and training data requirements remain significant challenges. <d-cite key="liang2024learning"></d-cite>

## Comparison and Synthesis
Although both papers improve robot behavior through natural-language feedback, they operate at different levels of the control stack and optimize different aspects of interaction. <d-cite key="shi2024yell,liang2024learning"></d-cite>

| Dimension          | YAY Robot                                                                                    | Language Model Predictive Control                                                          |
|:-------------------|:---------------------------------------------------------------------------------------------|:-------------------------------------------------------------------------------------------|
| What is corrected? | Selection of a language-conditioned physical skill or recovery instruction                   | LLM-generated robot program in a multi-turn teaching conversation                          |
| Human feedback     | Spoken intervention while a robot executes a task                                            | Natural-language instructions and corrective follow-ups after observing generated behavior |
| Fast adaptation    | A human instruction temporarily overrides the high-level policy                              | New dialogue context changes the next generated code response                              |
| Slow adaptation    | Fine-tuning the high-level instruction selector on saved corrections                         | Fine-tuning the code-writing LLM on successful or informative teaching trajectories        |
| Learned component  | High-level policy; low-level ACT motor controller stays frozen during correction fine-tuning | Code-generating LLM; LMPC-Rollouts additionally predicts dialogue continuations            |
| Primary objective  | Improve execution and recovery success for multi-step manipulation                           | Improve task success and reduce human correction turns (teachability)                      |
| Key limitation     | Cannot execute recovery skills absent from the low-level repertoire                          | Predicted interactions can be inaccurate and inference can be computationally costly       |

## Discussion

1. **Q: Is YAY Robot learning a general recovery strategy, or simply imitating the corrections provided by humans?**

   **A (class discussion):** There was no definitive way identified in the discussion to distinguish between these possibilities. Zero-shot evaluation on previously unseen failure situations was suggested as a way to investigate this question. If the robot can recover in those situations without additional human corrections, that would provide evidence that its learned behavior generalizes beyond the corrections seen during training.

2. **Q: In YAY Robot, is fine tuning a separate step from the live correction process? How?**

   **A (class discussion):** Yes. Live corrections are collected while the robot performs the task, and those corrections are then used in a separate fine-tuning step to improve the high level policy. The updated policy can then apply what it learned in future runs without requiring live human intervention.

3. **Q: What additional forms of human input could complement language instructions to help the robot understand more complex corrections?**

   **Discussion:** One suggestion from the class was to combine language instructions with human gestures captured through vision. For example, pointing could help indicate the direction in which the robot should move or identify the object a person is referring to. Combining verbal instructions with these visual cues could make directions clearer and provide a richer interface for more complex human instructions.
