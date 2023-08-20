---
title: "[Paper Review] Rethinking the Role of Demonstrations: What Makes In-Context Learning Work?"
excerpt: "Post displaying the various ways of highlighting code in Markdown."
last_modified_at: 2023-07-14
categories: 
  - Paper Review
tags: 
  - paper review
  - NLP
  - in-context learning
toc: true
toc_sticky : true
toc_label: "Contents"
use_math: true
---

<span style="color:gray"> [Rethinking the Role of Demonstrations: What Makes In-Context Learning Work?](https://arxiv.org/abs/2202.12837)", EMNLP 2022.
All figures are captured from the paper. </span>

## TL;DR

In-context learning does not necessarily requires ground truth input-label mapping. 
What really matters are 1) distribution of the input text, 2) the label space, 3) the format of demonstrations.

## In-Context Learning

In-context learning is a method of learning new tasks during inference that leads to the improved performance of downstream tasks without requiring additional parameter updates of model.
As shown in the figure below, a demonstration consists of several input-label examples, and the model learns a new task through this demonstration.

![figure1](https://github.com/hyeonjeong1/hyeonjeong1.github.io/assets/60830095/06fc6173-e2a9-4207-86a7-3bb251e0b90d){: width="420"}{: .align-center}

This paper investigates how the model learns and which elements of the demonstration impact the performance of downstream tasks.

## Experimental Setup

**<u>Model</u>** In this study, 6 language models were utilized and the names and sizes of the models are specified in below table. For each model, two inference methods, direct<sup id="a1">[1](#f1)</sup> and channel<sup id="a2">[2](#f2)</sup>, were applied. <br>

|Model|#Params|Public|Meta-trained<sup id="a1">[3](#f3)</sup>|
|:---|---:|---:|---:|
|GPT-2 Large|774M|O|X|
|MetaICL|774M|O|X|
|GPT-J|6B|O|X|
|fairseq 6.7B|6.7B|O|X|
|fairseq 13B|13B|O|X|
|GPT-3|175B|C|X|

**<u>Dataset</u>** The evaluation was conducted on 26 datasets, that cover various tasks and domains and include benchmark datasets studied from GLUE.

**<u>Other Details</u>** The demonstration was constructed using k=16 examples and minimal templates (e.g., separating input texts and labels using space or newline) by default.

## Ground Truth Matters Little
To investigate the importance of the model learning correct input-label pairs, the experiment was conducted with the following 3 different settings for inference:
- No demonstrations: typical zero-shot method, i.e., $\mathrm{argmax}_{y\in C} {P(y\vert x)}$.
- Demonstrations w/ gold labels: use correct input-label pairs, i.e., $\mathrm{argmax}_{y\in C} {P(y\vert x_1, y_1, \cdots, x_k, y_k, x)}$.
- Demonstrations w/ random labels: replace a gold label $y_i$ with a randomly sampled label $\tilde{y_i}$ from the label space $C$, i.e., $\mathrm{argmax}_{y\in C} {P(y\vert x_1, \tilde{y_1}, \cdots, x_k, \tilde{y_k}, x)}$ .

![result1](https://github.com/hyeonjeong1/hyeonjeong1.github.io/assets/60830095/4de75282-7aff-47b8-9b76-8debd428684b){: width="800"}{: .align-center}

From the above figure it can be observed that
1. Using the random label didn't result in a significant performance drop compared to using the gold label.
2. The performance of no demonstration case significantly dropped.

The model learns from demonstrations, but _it does not necessarily come from correctly paired input-label examples_.

## Why does In-Context Learning work?

If the gold label is not the primary factor influencing the performance, then _what aspects of the demonstrations does the model learn from_?

In addition to correct input-label pairs, the authors conducted experiments to investigate how the following three aspects impact performance:
- The distribution of the input text, i.e., the underlying distribution that $x_1, \cdots, x_k$ are form.
- The label space, i.e., the space covered by $y_1, \cdots, y_k$.
- The format-specifically, the use of input-label pairing as the format.

In this experiment, MetaICL shows a different tendency compared to other models, and this will be discussed later.

Impact of the distribution of the input text
To understand the impact of the distribution of input text, models were trained using out-of-distribution (OOD) text. Here, OOD text refers to unlabeled training data, so it can be understood as data that is different from the existing input text, i.e., outliers.

![figure8](https://github.com/hyeonjeong1/hyeonjeong1.github.io/assets/60830095/838ea210-c696-4852-bd5c-d5f4a49c8daa){: width="800"}{: .align-center}

As seen above, almost all models have significantly reduced performance, and even worse the performance than No demonstrations.
Thus, the in-distribution input text is crucial, as _the model inherently learns from in-distribution text during training_.

### Impact of the label space
Now, let's find out what impact a randomly extracted English word has on performance.
For this experiment, they construct $C_{rand}$ with $\vert C_{rand}\vert=\vert C\vert$, consisting of randomly selected English words. Then, a label $\tilde{y_i}\in C_{rand}$ is randomly paired with any existing input texts $x_i$.

![figure9](https://github.com/hyeonjeong1/hyeonjeong1.github.io/assets/60830095/27a47ddc-2144-47c7-b623-9e4836e06d90){: width="800"}{: .align-center}

Above figure shows that direct models were significantly influenced by the label space, resulting in a decrease in performance.
In contrast, channel models exhibited less impact from the label space.
In other words, the direct model, using $P(y|x)$ likelihood for inference, learns the distribution of the label space for a given input, while the channel model, using $P(x|y)$ likelihood for inference, does not learn from the distribution of the label space.

### Impact of the format
Here, "format" refers to the pairing of input and label.
Experiments were conducted by changing the format using methods such as demonstrations with no labels and demonstrations with labels only.

![figure10](https://github.com/hyeonjeong1/hyeonjeong1.github.io/assets/60830095/984c6183-c3a3-4fd5-8087-179e96129f81){: width="800"}{: .align-center}

According to above figure, when the format was changed (purple and green bars), the performance was close to or even worse than no demonstrations.
This can be understood as the format playing a crucial role in _providing hints to the model on how to mimic the desired inference_.

### Impact of the meta-training
 MetaICL shows negligible impact from random labels, but format changes, are using random labels only or using no labels, have more significant impact on the performance.
This can be understood as meta-training focusing on learning simple aspects such as format rather than correctly paired input-label examples from demonstrations.
In other words, the format is likely easier to exploit. <br>
Furthermore, the change in output space (i.e., Random English words) have little impact on Channel MetaICL unlike Direct MetaICL.
This indicates that exploiting the space of input text that model have to generate is easier than exploiting the space of input text that model conditions on.

## Conclusion
- Learning signals do not necessarily come from correctly paired input-label examples.
- Experimental results show that 1) distribution of the input text, 2) the label space, 3) the format of demonstrations are important.
- Meta-training magnifies these trends.

## References

[1] Min, Sewon, et al. "Rethinking the role of demonstrations: What makes in-context learning work?." arXiv preprint arXiv:2202.12837 (2022).


<b id="f1"><sup>1</sup></b> The conventional approach, calculate the likelihood of labels given inputs, i.e., $P(y \vert x)$.[↩](#a1)<br>
<b id="f2"><sup>2</sup></b> The approach that calculates the likelihood of inputs given labels, i.e., $P(x \vert y)$. Thus, given $y$, the model should explain all words in $x$.[↩](#a2)<br>
<b id="f3"><sup>3</sup></b> meta-training method with an in-context learning objective. For further details, please refer to [this paper](https://arxiv.org/abs/2110.15943).