---
layout: distill
title: "Non-verbal Interaction and Learning Signals"
description: "Implicit communication and learning signals: human gaze, head pose, facial expressions, and kinesic behaviors in interactive robotics."
date: 2026-10-19
future: true
htmlwidgets: true
ready: false

# Reporter team authors
authors:
  - name: "Angelica Bain, Srikar Bangaru, Samriddhi Kumar, Caroline Xu"

bibliography: 2026-10-19-non-verbal-interaction-and-learning-signals.bib

toc:
  - name: "Introduction"
  - name: "GIRAF"
  - name: "GABRIL: Gaze-Based Regularization for Mitigating Causal Confusion in Imitation Learning"
    subsections:
      - name: "Background and Motivation"
      - name: "Problem Formalization"
      - name: "Prior Work"
      - name: "Introducing GABRIL"
  - name: "Methodology"
    subsections:
      - name: "Spatiotemporal Gaze Mask"
      - name: "Gaze Based Regularization Architecture"
  - name: "Experimental Set Up"
  - name: "Results"
    subsections:
      - name: "Normal Atari Results"
      - name: "Confounded Atari Results"
      - name: "CARLA Result"
  - name: "Conclusion"
  - name: "Discussion"
    subsections:
      - name: "Questions about GABRIL"
      - name: "Questions relating to both papers"
  - name: "References"
---

### Introduction

As discussed in class before, there is often a **language grounding problem** for robots. That is, what is the actual process of connecting abstract natural language words, phrases, or instructions to physical, tangible elements in the real world. Asking a robot to bring an object may not describe or portray what the object is for. This inherently stems from ambiguous or incomplete sentences or or commands. Another aspect lies in imitation learning, where the robot only learns and does a specific behavior, not why they are doing it. This concept was also described as **causal confusion**, where an agent mistakenly relies on correlation rather than the true cause. This is why **non-verbal cues** are so important. Humans especially constantly use gestures, gaze, and body language to support or replace speech. These cues are natural and low effort, while also giving greater context open for interpretation. The main goal of this lecture is to provide ways for an agent to try and understand what a human means via non-verbal interactions.

# GIRAF

# GABRIL: Gaze-Based Regularization for Mitigating Causal Confusion in Imitation Learning

### Background and Motivation

We give robots a lot of training data, and it can do the right thing, but it does it for the right reason? Agents can learn from human expert demonstrations, but it is possible that they haven’t recognized the actual reason why. The presenters described this concept using the example of autonomous vehicles, where an autonomous car learns how to brake based on the brake light on the dashboard instead of actual traffic patterns/ pedestrians. This would lead to catastrophic failures in real world applications that are deployed, where agents learn misleading rules and wrong correlations. Why do we need to care about this problem? It can overfit to the brake indicator, but we want it to do the right thing. GABRIL aims to address the issue, known as **causal confusion**, by encouraging agents to focus on visual information that is relevant to the task. <d-cite key="banayeeanzade2025gabril"></d-cite>

### Problem Formalization

Imitation learning involves training a policy that maps observations into actions, which allows the agent to then reproduce those behaviors demonstrated by a human expert. That information contained in an observation can be split into two different categories: **causal factors and confounding variables**. **Causal factors** are essential for decision making, for example, recognizing what a traffic light is, or a pedestrian. **Confounding variables** are correlated with the observed action but they are not responsible for it. Standard **behavioral cloning** minimizes this prediction error between demonstrated and predicted actions, but doesn’t explicitly differentiate between the two categories. The ideal robust policy should remain invariant to changes in confounding variables, so the agent should make the same decision regardless of irrelevant features/ noise in the environment. <d-cite key="banayeeanzade2025gabril"></d-cite>

{% include figure.liquid 
   path="assets/img/2026-10-19-non-verbal-interaction-and-learning-signals/equation.png" 
   class="img-fluid rounded z-depth-1" 
   caption="Figure 1: Ideal Robust Policy." 
%}

### Prior Work

Previous research has tried to reduce casual confusion using methods like dropout, contrastive learning, and gaze based supervision. Dropout approaches, like OREO and GMD, selectively remove visual features using predefined heuristics or information on human gaze. However, these methods don’t have direct regularization to actively align the model’s focus. Other approaches, like contrastive learning and AGIL, modify input images by blurring or use gaze masks to guide learning. These techniques can introduce other challenges, however, as it can distort original input distributions/ training data or requires expensive gaze data during test time. These limitations only further motivate the development of a method like GABRIL where you can guide visual attention without having to modify the original input. <d-cite key="banayeeanzade2025gabril"></d-cite>

{% include figure.liquid 
   path="assets/img/2026-10-19-non-verbal-interaction-and-learning-signals/prior.png" 
   class="img-fluid rounded z-depth-1" 
   caption="Figure 2: OREO, Constranstive Learning, and AGIL approaches." 
%}

### Introducing GABRIL

The idea behind GABRIL is to introduce **latent space regularization**, that aligns the model’s internal feature maps with human gaze (like knowledge distillation), leaving raw inputs untouched. Additionally, it has **zero test-time overhead**, only requiring gaze data during offline training, meaning there is no additional gaze-tracking requirement during deployment. Finally, this approach can be seamlessly integrated with prior dropout methods, making it extremely complementary and combinable to improve imitation learning performance. <d-cite key="banayeeanzade2025gabril"></d-cite>

## Methodology

### Spatiotemporal Gaze Mask

GABRIL begins by converting raw eye tracking coordinates into **spatiotemporal gaze masks** that represent where a human focuses their attention. For each video frame, the recorded gaze coordinates are then used to construct a **2D Gaussian distribution** centered around the observed gaze location. This spatial representation models the area of visual attention instead of treating gaze as one single point. This method also has a temporal component because human visual attention is influenced by short term memory and info from nearby frames. These gaze samples from the past, present, and future are then accumulated using temporal decay factors and time scaled variance. Because this process happens during offline training, future gaze info can be incorporated without affecting real time execution. <d-cite key="banayeeanzade2025gabril"></d-cite>

{% include figure.liquid 
   path="assets/img/2026-10-19-non-verbal-interaction-and-learning-signals/spatio.png" 
   class="img-fluid rounded z-depth-1" 
   caption="Figure 3: Steps to build the Spatiotemporal Gaze Mask." 
%}

### Gaze Based Regularization Architecture

The authors proposed the GABRIL framework that is made up of an **image encoder** and an **action prediction network**. The image encoder transforms the original observation into an internal feature representation, and then the action predictor uses that representation to then determine the appropriate action. A separate **gaze prediction function** operates on the encoder’s features to estimate where a human would focus attention. The framework aims to help the agent identify meaningful features instead of relying on irrelevant corrections. <d-cite key="banayeeanzade2025gabril"></d-cite>

{% include figure.liquid 
   path="assets/img/2026-10-19-non-verbal-interaction-and-learning-signals/gaze.png" 
   class="img-fluid rounded z-depth-1" 
   caption="Figure 4: GABRIL Process." 
%}

The loss function of GABRIL combines both **behavioral cloning loss** along with the **gaze predictor loss**. The **behavioral cloning** class measures how accurately the model predicts expert actions, while the gaze predictor loss measures how closely the model’s predicted attention matches the human gaze information. Lambda, the weighing regularization parameter, controls the strength/ importance of the gaze based component. This combined approach encourages the model to both learn the correct actions and representations that align with human attention. <d-cite key="banayeeanzade2025gabril"></d-cite>

## Experimental Set Up

Researchers used 2 domains to evaluate GABRIL: **Atari games** and autonomous driving using the **CARLA simulator**. The Atari experiments had 15 games with 1,160 minutes of human expert demonstrations with eye tracking data. It was tested with both **normal and confounded environments**. One question that was brought up in lecture was what the confounded environment truly entailed. The team explained that the confounded environment had “noisy information” in the environment, such as numbers at the bottom of the screen, for example. The CARLA Autonomous Driving domain included 10 diverse urban driving tasks with 71 minutes of expert data and gaze tracking. One good point brought up in lecture is the stark difference between the minutes of data within the two Domains and what the potential implications of that means. The action space for this Domain was brake, steering angle, and throttle. The metric for the Atari domain was advantageous over BC. The metrics for CARLA included route completion (%) and **infraction penalty**. Completion was the percentage of error free journey, while penalties included events such as collusions, running red lights, etc. <d-cite key="banayeeanzade2025gabril"></d-cite>

{% include figure.liquid 
   path="assets/img/2026-10-19-non-verbal-interaction-and-learning-signals/setup.png" 
   class="img-fluid rounded z-depth-2" 
   caption="Figure 5: Top - Expert gaze guides to causal factor. Bottom - Normal and Confounded Atari Games environment." 
%}

## Results

### Normal Atari Results

The results for normal Atari environments showed that GABRIL improves imitation learning performance, specifically when combined with existing dropout based methods. Standard **behavioral cloning** was compared with other architecture based and regularization based methods. The strongest results were brought by combining GABRIL with dropout methods like OREO or GMD. According to the results table, the **GMD and GABRIL** combination achieved a mean ABC of around 22.7% while **OREO and GABRIL** achieved around 21.99%. These findings suggest that gaze based regularization can complement other learning strategies and help agents identify more useful visual information. <d-cite key="banayeeanzade2025gabril"></d-cite>

{% include figure.liquid 
   path="assets/img/2026-10-19-non-verbal-interaction-and-learning-signals/normal.png" 
   class="img-fluid rounded z-depth-1" 
   caption="Figure 6: Normal Atari Results." 
%}

### Confounded Atari Results

For the confounded Atari environment, we can see that GABRIL with the dropout based methods, especially GMD have the best result, with a **32.7% mean ABC**. These experiments were especially important because they examined the **causal confusion** problem. We can see that contrastive learning should relatively poor performance in the confounded settings, so combining these methods with human gaze can reduce the overall reliance on environmental noise. <d-cite key="banayeeanzade2025gabril"></d-cite>

{% include figure.liquid 
   path="assets/img/2026-10-19-non-verbal-interaction-and-learning-signals/confounded.png" 
   class="img-fluid rounded z-depth-1" 
   caption="Figure 7: Confounded Atari Results." 
%}

We can see that generally, GABRIL performed better as the proportion of available gaze data increases, indicating that more data is definitely needed to train and produce better results. <d-cite key="banayeeanzade2025gabril"></d-cite>

{% include figure.liquid 
   path="assets/img/2026-10-19-non-verbal-interaction-and-learning-signals/scale.png" 
   class="img-fluid rounded z-depth-1" 
   caption="Figure 8: Trend of GABRIL." 
%}

### CARLA Result

In the CARLA environment, it is seen that the combination of GABRIL and GMD showed very high, competitive results. This was both on seen and unseen routes during the method. GABRIL on its own also produced good results, however not as good as the previous combination mentioned. For the interpretability of GABRIL in CARLA, it is important to note that GABRIL was seen to direct its attention toward important features of the driving environment, such as pedestrians and other relevant objects, more consistently than standard **behavioral cloning**. Although GABRIL’s attention didn’t perfectly mimic the human gaze, it was more closely aligned with task relevant regions. Contrastly, behavior cloning sometimes focused on irrelevant parts of the scene. <d-cite key="banayeeanzade2025gabril"></d-cite>

{% include figure.liquid 
   path="assets/img/2026-10-19-non-verbal-interaction-and-learning-signals/carla.png" 
   class="img-fluid rounded z-depth-1" 
   caption="Figure 9: Carla Results." 
%}

## Conclusion

In conclusion, GABRIL demonstrates that human eye gaze can be used to reduce **causal confusion** in imitation learning by guiding the model's internal visual attention. The method introduces a regularization loss that aligns feature activations with **spatiotemporal gaze masks**, helping the agent focus on relevant visual information without altering the original input images. One of its main strengths is that eye tracking data is required only during training, resulting in no additional gaze related overhead during deployment. The method can also be combined with existing regularization techniques to improve performance. However, GABRIL depends on the **quality of expert gaze data**, which may introduce biases and increase data collection costs. The presentation also identifies the need to investigate different regularization weights and suggests extending the approach beyond vision to incorporate additional modalities such as audio or temperature. <d-cite key="banayeeanzade2025gabril"></d-cite>

## Discussion

The following includes the discussion questions presented by the team, along with the class discussion that occurred during the presentation.

### Questions About GABRIL

**One problem with regularization method is the robustness to noisy supervision, how should we deal with this obstacle?**

In class we discussed that one of the main issues is that we have limited human data, and that humans naturally make errors/ mistakes. Thus, one point that a classmate bought up is that we must also train robots to be more robust against human error. This could be through the form of adding more bias, or just any way that we can detect these issues in the model so we can avoid the risk of it. It could be useful to have a way to detect these errors within the model so we can reduce the buildup of risks. This could be part of the reason why GABRIL uses a temporal Gaussian approach, as was brought up in discussion, since it helps account for the natural noise in eye gaze data.

**How can GABRIL be generalized to multi-modal approaches?**

As discussed in class, one way that GABRIL can be generalized to multi modal approaches is adding more sensors to the model. Normally, when we have the model we have a few sensors that we collect data from, but instead of only relying on eye gaze sensors, we can add more to collect different types of information from the relevant sensors. For example, as a classmate brought up in class, adding tactile sensors could help the robot better understand its physical environment and how it interacts with objects, while a time formation sensor could be used to help map out spatial formations.



### Questions Relating to Both Papers

**Could there be a single system that uses gestures and eye gaze together, like using gaze to disambiguate a vague motion made by a human? If this is possible, when these two signals disagree, how should the robot decide which signal to trust about the human intent?**

One point that the class brought up for this question was that it shouldn’t be too much of a hard choice. It seems possible to combine gestures and eye gaze into a single system, as they both provide useful info about human intent. For example, we could take two inputs and from there distinguish which one we should trust more, or even how to combine the inputs. This would relate to **sensor fusion**, where info from multiple areas is combined to help the robot make a better decision.

**Social cues and gestures are not completely universal; there are signals that differ between cultures (some can even be considered offensive). Should this be incorporated into the context the robot receives?**

One consensus that the class had was that yes, there are definitely signals that differ between cultures, and eventually, we want it to be incorporated into the context that the robot receives. For example, the presenters gave the example of nodding your head vs moving it side to side - in some countries the first movement would be “yes”, the second “no”. However, in others, the same actions might be flipped. Another classmate brought up the example of how they would personally prefer not to be flipped off by the robot - a completely valid take. In general, the consensus was that these cultural differences should eventually be incorporated into the context that the robot receives, and eventually with more training data from diverse perspectives, it can pick up not only the context, but figure out who it is interacting with. Once again, this is hard with less data.
