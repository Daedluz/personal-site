+++
date = '2025-08-12T01:16:16-04:00'
draft = false
title = 'Membership Inference Attack with Cumulative Topological Distance'
+++

# Introduction
As machine learning models become increasingly powerful and pervasive, concerns about privacy in their deployment have grown. One notable risk comes from **membership inference attacks**, where an adversary attempts to determine whether a particular data point was used to train a given model. For example, in healthcare, a membership inference attack could reveal if someone participated in a clinical study, threatening patient confidentiality.
This risk motivates my work in this post, where I document my recent experiment investigating the possibility in using cumulative topological distance in membership inference attacks. 

# Methodology
## Preliminaries
In this section, I'll briefly explain how membership inference attacks and TED-LaST works
### Membership Inference Attacks
For a target model, the attacker would train a *shadow model* on a auxiliary dataset that belongs to the same data distribution as the target model's training set.

Then attacker would obtain the logits for member and non-member data of the shadow model. After that, the attacker would construct an *attack dataset* where $x$ is the logits obtained, and $y$ is the membership status.

A *attack model* would then be trained on the attack dataset to distinguish between the logits of member and non-member samples.

### TED-LaST
This is a method designed to detect backdoor samples in poisoning attacks, we will focus on the part of cumulative topological distance(ctd). 

First, it assumes that the defender has access to a set of clean sample (without trigger and correctly labeled).
For each sample in the clean subset, obtain the activations of from each layer, then construct a K-nearest neighbor model for each layer.

Next, for a sample $x$ that we want to calculate ctd, we first obtain its activation from each layer, denoted as $x_l$. We'll then get k nearest neighbor for $x_l$ and calculate the rank of it's nearest neighbor from the same class as $K_l$.

Below is as example:
![400](https://i.imgur.com/3PuCmdi.jpeg)

We take $x_l$ as an example, we iterate through its k nearest neighbors $n_{1...k}$, see if $n_i$ share the same label as $x$, the rank of that neighbor is $i-1$. After we find the nearest neighbor, we record it's rank as $K_l$.

Ultimately, we will have a vector $K$. $K$ represents the topological evolution of a sample when it makes its way through the neural network.

The author also stated that each layer should be assigned different weights, but we will not cover the detail of weight calculation in this post.

## Overview
After reading TED-LaST, I developed the idea of using CTD as a membership inference attack feature. My original hypothesis is if a model is trained on an image, it would recognize it and thus having a smaller CTD score compared to those the model are not trained on.

So basically the idea is to replace logits in traditional membership inference attacks into ctd values. 

For the attack model, we considered two scenarios:
- Train a pca-based outlier detector on the ctd values for non-members, then classify the outliers as members
- Train a multi-layer perceptron to distinguish between member and non-members 

Note that based on the nature of ctd, our attack needs to obtain activations from each layer of the neural network, making it a white-box attack. So we would expect a higher performance due to the extra information given.

# Experiment
## Experiment Setup
I used ResNet 10 for both target and shadow model.
The dataset used in this experiment is **CIFAR-10**, there's 60000 images in this dataset, which is subsequently being divided into:
- Target model training (25000)
- Target model validation (5000)
- Shadow model training (15000)
- Shadow model testing (10000)
- KNN samples (5000)
## Result

###### Outlier Detector
![500](https://i.imgur.com/7VhZkrs.png)

###### MLP
![500](https://i.imgur.com/f21wnvO.png)

Well, as we can see in the ROC curve, this attack is about the same as random guessing...
# Discussion
TED-LaST is originally designed for detecting poisoning attacks. The clean subset that are used to construct KNN act as ground truth answers that are not poisoned. Then we can view cumulative topological distance as a value representing "how misclassified an image is". In TED-LaST, this value is calculated for each layer in the neural network.

Next, we talk a little bit about adaptive attacks that TED-LaST tried to tackle with. In adaptive attacks, the attacker uses a technique called "laundry", where the attack produces samples that **contains the trigger but with correct label**. This technique forces the model to associate the trigger with both the target class and its original class, making the trigger a "feature" the model considers when classifying an image. This trait makes the model prone to misclassifying the image when the image goes through the model, primarily between target label and the image's original label. Since both classes contain this trait (poisoned samples and regularization samples).

However, in membership inference attacks, the images does not contain triggers.

Sure there might be images that share similar traits (like cats and dogs both could have pointed ears), but this trait is consistent across all samples from the same class, whereas for backdoor detection, the trigger is only present in a **subset of images**, making that specific proportion of images more prone to misclassification as the sample goes through the model, resulting in a higher ctd values.

In the case of regular classification, all samples from the same class would have a similar cumulative topological distance, no matter if it's a member or non-member, making this method ineffective on membership inference attacks.
