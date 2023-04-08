---
title: "Density Ratio Estimation and Neyman Pearson Classification with Missing Dara"
collection: publications
permalink: /publication/DRE&NPwithMissingData
excerpt: 'This paper adapts Density Ratio Estimation techniques making them robust to missing not at random missing data before applying this to the field of Neyman Pearson classification.'
date: 2023-04-24
venue: 'AISTATS 2023'
paperurl: 'https://arxiv.org/abs/2302.10655'
citation: 'Your Name, You. (2009). &quot;Paper Title Number 1.&quot; <i>Journal 1</i>. 1(1).'
---
Density Ratio Estimation (DRE) is an important machine learning technique with many downstream applications. We consider the challenge of DRE with missing not at random (MNAR) data. In this setting, we show that using standard DRE methods leads to biased results while our proposal (M-KLIEP), an adaptation of the popular DRE procedure KLIEP, restores consistency. Moreover, we provide finite sample estimation error bounds for M-KLIEP, which demonstrate minimax optimality with respect to both sample size and worst-case missingness. We then adapt an important downstream application of DRE, Neyman-Pearson (NP) classification, to this MNAR setting. Our procedure both controls Type I error and achieves high power, with high probability.
Finally, we demonstrate promising empirical performance both synthetic data and real-world data with simulated missingness.

[Download paper here](https://arxiv.org/abs/2302.10655). 
[Code repository here](https://github.com/joshgivens/DRE-NP-MissingData)