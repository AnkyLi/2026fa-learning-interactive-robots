---
layout: distill
title: "Intent/Trajectory Prediction"
description: "Forecasting human motion and navigation trajectories in shared physical environments using generative and sequence models."
date: 2026-10-26
future: true
htmlwidgets: true
ready: false

# Reporter team authors
authors:
  - name: "Vu-Anh Le"
  - name: "Jameson Sandler"
  - name: "Anqi Li"
  - name: "Sudeesh Murugesan"

bibliography: 2026-10-26-intent-trajectory-prediction.bib

toc:
  - name: "Introduction"
  - name: "Social GAN: Socially Acceptable Trajectories with Generative Adversarial Networks"
  - name: "Estimating Cognitive Biases with Attention-Aware Inverse Planning"
  - name: "Comparison and Discussion"
  - name: "Conclusion"
  - name: "References"
---

## Introduction

Predicting human intent and future trajectories is an important problem in human-robot interaction. When robots work in environments with people, they need to consider not only their own actions but also how people around them may behave. For example, a delivery robot moving through a crowded area needs to predict pedestrian movements to avoid collisions and navigate safely. However, human behavior is difficult to predict because people can have different goals, interact with each other, and make decisions based on incomplete information.

This report discusses two papers that study human behavior prediction from different perspectives. The first paper, Social GAN: Socially Acceptable Trajectories with Generative Adversarial Networks, focuses on predicting multiple possible pedestrain trajectories by modeling social interactions. The second paper, Estimating Cognitive Biases with Attention-Aware Inverse Planning, considers how limited attention and cognitive biases affect human decisions. Instead of assuming that people always make optimal decisions bases on complete information, this approach tries to infer what people may attend to or ignore from their observed behavior.

Although the two papers use different methods, both address important challenges in predicting human behavior. Social GAN focusses on generating diverse and socially acceptable future trajectories, while attention-aware inverse planning provides a way to explain behavior through underlying attentional biases. In this report, we review their main approaches, experimental results, and limitations, and discuss how these ideas may help robots interact more safely with humans.

## Social GAN: Socially Acceptable Trajectories with Generative Adversarial Networks

### Motivation and Problem

Predicting pedestrian movement in crowded environments is challenging because people do not move independently. Their future paths depend on both their previous movements and the behavior of others around them. For example, when two pedestrian approach each other, they may change direction, slow down, or choose different paths to avoid a collision. These different behaviors can all be reasonable, which means that predicting only one future trajectory may not be sufficient.

Previous methods have used hand-crafted features or recurrent neural networks to model pedestrian movements. However, many of these approaches mainly consider interactions within a local neighborhood. Another limitation is that minimizing the L2 prediction error may encourage the model to learn an average trajectory rather than multiple possible future paths.

To address these problems, Gupta et al. proposed Social GAN, which uses generative adversarial learning to predict diverse and socially acceptable pedestrian trajectories. The method also introduces a pooling mechanism to capture interactions between pedestrians and a variety loss to encourage different predictions for the same observed history <d-cite key="gupta2018social"></d-cite>.

### Methodology
Social GAN aims to predict the future trajectories of all pedestrians in a scene based on their observed movements. For pedestrian $i$, the observed trajectory is represented as 

$$
X_i = \{(x_i^t, y_i^t)\}_{t=1}^{t_{\mathrm{obs}}}
$$

where $(x_i^,y_i^t)$is the position at time $t$. The model then predicts the future trajectory $\hat{Y}_i$ from $t_{obs}+1$ to $t_{pred}$.

The model consists of three main components: a Generator, a Pooling Module, and a Discriminator, as shown in Figure 1.

{% include figure.liquid 
   path="assets/img/2026-10-26-intent-trajectory-prediction/sgan_architecture.png" 
   class="img-fluid rounded z-depth-1" 
   caption="Figure 1: Social GAN architecture, showing the generator, pooling module, and discriminator. Adapted from Gupta et al. (2018), Figure 2." 
%}

The Generator uses an LSTM-based encoder-decoder architecture. The encoder processes the observed positions of each pedestrian and produces hidden states representing their movement history. The Pooling Module then combines information from pedestrians in the scene to capture their interactions. The decoder uses the encoded history, pooled information, and a random noise vector to generate future positions. By sampling different noise vectors, the generator can produce different future trajectories for the same observed history.

The proposed pooling mechanism considers the relative positions and hidden states of all pedestrians instead of only using a fixed local neighborhood. These features are processed by a multilayer perceptron (MLP) and combined using max pooling. This produces a fixed-size representation that summarizes information from other pedestrians and helps the model learn social interactions.

The Discriminator is another LSTM-based network. It receives a complete trajectory consisting of the observed history and either the real future path or a generated prediction. It then classifies whether the trajectory is real or generated. Through adversarial training, the generator learns to produce trajectories that are more similar to observed pedestrian movements.

To improve prediction diversity, the authors introduce a variety loss. For each observed history, the generator produce $K$ possible future trajectories using different random noise samples. The loss is calculated using the generated trajectory closest to the ground truth:

$$
\mathcal{L}_{variety}
=
\min_{K \in \{1, \ldots,K\}}
\left\|Y_i-hat{Y}_i{(k)}\right\|_2.
$$

Here, $Y_i$ is the ground-truth future trajectory, $\hat{Y}_i^{(k)}$ is the $k$-th generated trajectory, and $K$ is the number of sampled predictions. This loss encourages the model to generate different possible trajectories rather then making every prediction close to one average path. The variety loss is combined with adversarial training to improve both diversity and prediction accuracy.

### Experiments and Results

The authors evaluated Social GAN using the ETH and UCY pedestrian trajectory datasets, including five subsets: ETH, HOTEL, UNIV, ZARA1, and ZARA2. These datasets contain real pedestrian movements in different environments. The model observes 8 times steps (3.2 seconds) and predicts the next 8 or 12 time steps (3.2 or 4.8 seconds). The experiments use a leave-one-out setting, where four subsets are used for training and the remaining subset is used for testing.

The prediction accuracy is evaluated using two metrics:

- **Average Displacement Error (ADE):** The average Euclidean distance between predicted and ground-truth positions across all future time steps.
- **Final Displacement Error (FDE):** The Euclidean distance between the predicted final position and the actual final position.

Lower ADE and FDE values indicate more accurate predictions.

The authors compared Social GAN with several baseline methods, including Linear prediction, LSTM, and Social LSTM. Table 1 presents the ADE and FDE results on the five pedestrian datasets.

{% include figure.liquid
   path="assets/img/2026-10-26-intent-trajectory-prediction/sgan_results.png"
   class="img-fluid rounded z-depth-1"
   caption="Table 1: Quantitative comparison of trajectory prediction methods using ADE and FDE. Reproduced from Gupta et al. (2018), Table 1."
%}

*Table 1: Average ADE and FDE (meters) for 8_step and 12-step predictions, adapted from Gupta et al. (2018), Table 1. Lower values are better.*

The results show that SGAN-20V-20 achieves the lowest average ADE and FDE among the selected methods. The full model with pooling, SGAN-20VP-20, has slightly higher errors, but still performs better than the LSTM and Social LSTM baselines on average. The Scocial GAN results are based on selecting the best prediction from 20 generated samples during evaluation, while the baseline models produce a single prediction.

The authors also conducted ablation experiments to study the effects of variety loss and social pooling. The results show that variety loss improves prediction accuracy by encouraging the model to generate more diverse possible futures. Simply generating more samples from a model trained without variety loss does not provide the same improvement.

Although pooling does not produce the lowest displacement errors, it helps the model generate more socially acceptable trajectories. Figure 2 compares Social GAN with and without pooling in different pedestrian interaction scenarios.

{% include figure.liquid
   path="assets/img/2026-10-26-intent-trajectory-prediction/sgan_collision_avoidance.png"
   class="img-fluid rounded z-depth-1"
   caption="Figure 2: Comparison of Social GAN predictions without pooling (top) and with pooling (bottom) in collision avoidance scenarios. Reproduced from Gupta et al. (2018), Figure 5."
%}

The qualitative results show that the pooling-based model can better represent behaviors such as changing direction to avoid other pedestrian, moving around a group, and slowing down to let another person pass. The authors also report that the pooling-based model is approximately 16 times faster than Social LSTM in their benchmark, mainly because it avoids performing the same expensive pooling operations at every time step.

### Limitations and Takeaways

Social GAN shows that pedestrian trajectory prediction can benefit from modeling both social interactions and multiple possible future movements. Its main contributions include the GAN-based prediction framework, the pooling mechanism for capturing interactions, and the variety loss generating diverse trajectories.

However, the method mainly focuses on pedestrian movement and does not explicit model envioronmental obstacles, scene samentics, or long-term human goals. Its prediction performance also depends on the observed trajectories of surrounding pedestrains, which may be incomplete or noisy in real-world applications.

Another limitation is the best-of-multiple-samples evaluation. Although generating several possible trajectories improves ADE and FDE, a robot cannot know in advance which prediction will match the autual future movement. Therefore, additional planing methods would be needed to use these predictions safely in real robot navigation.

Overall, Social GAN provides an approach for predicting multiple plausible pedestrian trajectories instead of relying on a single future path. It also shows why modeling social interactions is important for robots navigating in environments share  with humans.
