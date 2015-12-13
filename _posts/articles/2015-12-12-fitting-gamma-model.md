---
layout: post
title: "Fitting a Gamma model to data using Metropolis Hastings"
modified:
categories: articles
excerpt:
tags: []
image:
  feature:
date: 2015-12-12
---

Keynote Systems measures download times for selected web sites.  For example, consider this data: [Keynote Systems download times](http://www.keynote.com/keynote_competitive_research/performance_indices/business_index/business.html)

We will use a gamma likelihood for this dataset with both gamma parameters unknown.  We'll use exponential priors for both parameters and since we don't have much intuition about these parameters, we'll make them a bit vague, picking both to be exponential with mean 10.  

Now, we want to fit this model by drawing a posterior sample using the Metropolis-Hastings algorithm.  First, we will need to compute the acceptance probabilities.  So, in our gamma model the download times \\(y_i\\) are distributed like

\\[y_i \sim Gamma(\alpha,\beta)\\]

So, the probability of a download time, conditional on the gamma parameters, is given by

\\[P(y_i \| \alpha,\beta) = \frac{\beta^\alpha}{\Gamma(\alpha)}y_i^{\alpha -1}e^{-\beta y_i}\\]

The joint likelihood of \\(\alpha\\) and \\(\beta\\) given the sample of \\(n\\) download times is thus

\\[
L(\alpha,\beta \| \underset{\sim}y) = f(\underset{\sim}y \| \alpha,\beta) = \left(\frac{\beta^\alpha}{\Gamma(\alpha)}\right)^n\prod_{i=1}^ny_i^{\alpha-1}\left(e^{-\beta\sum\limits_{i=1}^n y_i}\right)
\\]

With priors for \\(\alpha\\) and \\(\beta\\):  \\(P(\alpha) = \frac{1}{10}e^{\frac{\alpha}{10}}\\) and \\(P(\beta) = \frac{1}{10}e^{\frac{\beta}{10}} \\) we have a posterior probability:

\\[
\pi(\alpha,\beta \| \underset{\sim}y) = \left(\frac{\beta^\alpha}{\Gamma(\alpha)}\right)^n\prod_{i=1}^ny_i^{\alpha-1}\left(e^{-\beta\sum\limits_{i=1}^n y_i}\right) \frac{1}{10}e^{\frac{\alpha}{10}} \frac{1}{10}e^{\frac{\beta}{10}}
\\]

Since  \\(P(\beta) = \frac{1}{10}e^{\frac{\beta}{10}} \\)  is a constant with respect to \\(\alpha\\), the acceptance probability for \\(\alpha\\) is

\\[
a_{\alpha} = \frac{\pi(\alpha^*,\beta|\underset{\sim}y)}{\pi(\alpha_i,\beta\|\underset{\sim}y)}=
\\]