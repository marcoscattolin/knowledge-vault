---
title: "A Quick Uplift Modeling Introduction"
source: "https://medium.com/data-science/a-quick-uplift-modeling-introduction-6e14de32bfe0"
author:
  - "[[Shelby Temple]]"
published: 2020-06-27
created: 2026-04-17
description: "An introduction to Uplift Modeling for data science applications. Uplift modeling improves upon classic data science modeling approaches."
tags:
  - "clippings"
---
## [TDS Archive](https://medium.com/data-science?source=post_page---publication_nav-7f60cf5620c9-6e14de32bfe0---------------------------------------)

[![TDS Archive](https://miro.medium.com/v2/resize:fill:38:38/1*JEuS4KBdakUcjg9sC7Wo4A.png)](https://medium.com/data-science?source=post_page---post_publication_sidebar-7f60cf5620c9-6e14de32bfe0---------------------------------------)

An archive of data science, data analytics, data engineering, machine learning, and artificial intelligence writing from the former Towards Data Science Medium publication.

## Learn how uplift modeling can improve classic data science applications.

![](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*0y6ObDZF5meMoLFnEzru1w.jpeg)

Shutterstock: apixelstudio

## Introduction

*This article covers the ideas behind uplift modeling.*

By the end of this article, you will be comfortable explaining what uplift modeling is to someone else.

This is the first article in an uplift modeling collection.

1. Uplift Modeling: A Quick Introduction
2. Applied Uplift Modeling Example with Python (coming soon)
3. Lead Optimization Example with Uplift Modeling (coming soon)
4. Comparison of Different Uplift Techniques (coming soon)
5. 5 Tips for Uplift Modeling (coming soon)
6. End-to-End Uplift Modeling Project (coming soon)

## Problem Introduction

*We start with a real application then later generalize how uplift modeling can be useful for any industry and business unit.*

An insurance company is allocating new leads to insurance agents from the prior day based on the order they were gathered. Via outbound phone call campaigns agents can convert **5%** of worked leads to sales.

The company has realized they now generate more leads in a day than agents are able to work. There is also a sense that a lot of the leads are a waste of time.

With a desire to be more data-driven, the company’s data science team wants to optimize the order in which leads are worked. Using the lead data collected from the website, they built a model scoring the probability each lead was likely to convert to a sale.

This way an agent can start each day paying close attention to leads that have the best chance to convert to a sale in the future. They can also avoid wasting time on leads that will most likely never result in a sale.

Once the model was put into production the company observed that agents were now converting **10%** of worked leads into sales. That is 100% improvement over the old operation!

We need to hit the breaks! No high-fives just yet…

## Create an account to read the full story.

The author made this story available to Medium members only.  
If you’re new to Medium, create a new account to read this story on us.

[Continue in app](https://play.google.com/store/apps/details?id=com.medium.reader&referrer=utm_source%3Dregwall&source=-----6e14de32bfe0---------------------post_regwall------------------)

Or, continue in mobile web

Already have an account? [Sign in](https://medium.com/m/signin?operation=login&redirect=https%3A%2F%2Fmedium.com%2Fdata-science%2Fa-quick-uplift-modeling-introduction-6e14de32bfe0&source=-----6e14de32bfe0---------------------post_regwall------------------)