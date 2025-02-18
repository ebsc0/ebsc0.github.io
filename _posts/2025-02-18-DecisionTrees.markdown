---
layout: post
title: "Decision Trees"
date: 2025-02-17 18:02:00 -0400
category: project
---

Decision Trees, often referred to as Classification and Regression Trees (CART), partition the input space recursively into smaller, more homogeneous regions. Each region is associated with a simple model (e.g., a constant value in regression or a class distribution in classification). This tree-like model makes decisions by applying a series of rules on feature values, ultimately arriving at a leaf node containing the final prediction.

## Definition

A Decision Tree is defined by splitting the dataset at various internal nodes according to rules such as "feature $$x_j$$ < threshold $$t$$" (for continuous features) or "feature $$x_j$$ = value $$v$$" (for categorical features). The splits continue recursively until the data in each region is as pure as possible for the prediction task. In classification tasks, each leaf corresponds to a distribution over class labels; in regression tasks, each leaf contains a constant prediction, often the mean of the target values falling into that leaf.

For an input $$(x)$$ and parameters $$(\theta)$$, a regression tree is formally defined as:

$$
f(x;\theta) = \sum_{j=1}^{J}w_j\mathbb{1}(x \in R_j)
$$

where $$R_j$$ is the region corresponding to the $$j$$-th leaf, $$w_j$$ is the leaf's prediction (e.g., the average response for observations in $$R_j $$), and $$J$$ is the total number of leaf nodes.

The summation over $$J$$ is only to choose for some $$j$$ as the indicator function is only 1 when $$x$$ is in region $$R_j$$.

## Why Use Decision Trees?

- **Interpretability**: Trees provide human-readable decision paths, mimicking how people logically break down decisions.
- **Variable Selection**: By examining which features are used at top splits, one can gauge feature importance.
- **Mixed Data Types**: Decision Trees handle both numerical and categorical data without additional preprocessing.
- **Handling Missing Data**: Through methods like surrogate splits, trees can handle missing features in a relatively ad hoc but often effective manner.
- **Non-Linear Boundaries**: Axis-aligned splits allow Decision Trees to capture complex, non-linear decision surfaces.

## Model Fitting

We must minimize the loss function:

$$
\mathcal{L}(\theta) = \sum_{n=1}^{N}\ell(y_n, f(x_n;\theta)) = \sum_{j=1}^{J} \sum_{x_n \in R_j}\ell(y_n,w_j)
$$

where the loss function can be simplified since a regression tree’s prediction $$f(x_n;\theta)$$ is constant (i.e., $$w_j$$) in each leaf $$j$$, we can rewrite the total loss by grouping points according to the leaf they fall into where each leaf $$j$$ corresponds to a region $$R_j$$, all points $$x_n$$ in $$R_j$$ get the same predicted value $$w_j$$ so for those points, the loss is $$\ell(y_n, w_j)$$.

Hence, the double sum says: “For each leaf $$j$$, sum the loss for every training point $$x_n$$ in $$R_j$$. Then sum that over all leaves.”

Unfortunately, the loss function is not differentiable and finding the optimal partitioning of data is NP-complete. Therefore, we must use a greedy algorithm to try one node at a time.

## Splitting Criteria

At each node, the learning algorithm searches over all possible features (and thresholds, if numeric) to find the split that yields the greatest "purity" improvement. Common ways to measure the "purity" or reduction in impurity include:

- **Gini Impurity (Classification)**:

  $$
  G_i = \sum_{c=1}^{C} \hat{\pi}_{ic}(1 - \hat{\pi}_{ic}) = \sum_{c}\hat{\pi}_{ic} - \sum_{c}\hat{\pi}_{ic}^{2} = 1 - \sum_{c} \hat{\pi}_{ic}^{2}
  $$

  where $$\hat{\pi}_{ic}$$ is the proportion of samples in class $$c$$.

- **Entropy (Classification)**:

  $$
  H_i = \mathbb{H}(\hat{\pi}_i) = -\sum_{c=1}^{C} \hat{\pi}_{ic} \log\hat{\pi}_{ic}
  $$

- **Mean Squared Error (Regression)**:

  $$
    \text{cost}(D_i) = \frac{1}{|D|}\sum_{n \in D_i} (y_n - \bar{y})^2
  $$

  where $$y_n$$ is the predicted value for the $$n$$-th observation.

## Overfitting and Pruning

Decision Trees can grow until every leaf is pure or contains very few samples, often leading to overfitting. Although a tree can achieve extremely low training error, it may fail to generalize well.

To mitigate this:

1. **Pre-Pruning** (early stopping): Stop splitting when a node contains fewer than a certain number of samples, or when the improvement from a split falls below a threshold.
2. **Post-Pruning** (cost complexity pruning): Grow a large tree, then prune subtrees by merging leaves from the bottom up. One approach defines a complexity-penalized risk:

   $$
      R_{\alpha}(T) = R(T) + \alpha |T|
   $$

   where $$R(T)$$ is the training error, $$T$$ the number of leaves, and $$\alpha$$ a parameter controlling model complexity.

## Strengths and Limitations

**Strengths:**

- Easy to interpret and visualize.
- Minimal data preprocessing required.
- Can handle heterogeneous feature types.
- Reasonably fast to train and predict.

**Limitations:**

- High variance: small data changes may lead to large structural changes.
- Easily overfits if grown too deep without pruning.
- May not always yield the best predictive accuracy compared to more sophisticated models (e.g., ensembles).
