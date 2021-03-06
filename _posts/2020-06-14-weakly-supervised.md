---
title: 'Paper: End-to-end weakly-supervised semantic alignment'
date: 2020-06-14
permalink: /posts/2020/06/14/weakly-supervised
tags:
  - machine learning
  - preprocessing
  - weakly upervised
---
[PDF](http://openaccess.thecvf.com/content_cvpr_2018/papers/Rocco_End-to-End_Weakly-Supervised_Semantic_CVPR_2018_paper.pdf)  
**Year**: 2018  
**Conference**: CVPR  
**Goal**:Aligning two mages depicting objects of the same category  







Note: Parts of the paper may be quoted without indication.  

# Paper Resume: End-to-end weakly-supervised semantic alignment  
## Contributions:   
* End-to-end CNN 
* Weakly supervised (image pairs)
* differentiable soft inlier scoring module, inspired by the RANSAC  
## Introduction:  
In this work authors study the problem of **finding category-level correspondence**,
or semantic alignment, where the goal is to establish **dense correspondence between different objects belonging to the same category**.
This is also an extremely challenging task because of the
large intra-class variation, changes in viewpoint and presence of background clutter.  

The work is supposed to address previous limitations:
* the image representation and the geometric alignment model are not trained
together in an end-to-end manner.  
* Supervised methods require strong supervision in the form of ground truth correspondences.  

The outcome is that the image representation can be
trained from rich appearance variations present in different
but semantically related image pairs.

![Example](https://camo.githubusercontent.com/315c1bcefc0db56ac1d0d25ffbb5896bcac80fd1/687474703a2f2f7777772e64692e656e732e66722f77696c6c6f772f72657365617263682f7765616b616c69676e2f696d616765732f7465617365722e6a7067)

## Core Idea  
Here we can see the core method proposed in the paper:  
![img](/images/papers/weakly.PNG)  

The network is trained using pairs of images. It estimates a transformation to align them. At the very end, the quality of the alignment is estimated. The network is trained to maxize that quality without using strong supervision.  

### Feature Extractor  
The input is a set of images $$(I^s,I^t)$$ which pass through a siamese network. The output shape is a 2D grid of shape HxW with d channels. Each point of the grid is called $$f_{ij} \epsilon R^d$$  

## Pairwise feature matching  
Pairwise similarities--> normalized correlation function:  
$$S:{\Bbb R}^{h\times w\times d}\times{\Bbb R}^{h\times w\times d} \rightarrow {\Bbb R}^{h\times w\times  h\times w}$$  
$$s_{ijkl}=S(f^s,f^t)_{ijkl}=\frac{ \langle f^s_{ij:},f^t_{kl:} \rangle }{\sqrt{\sum_{a,b}  \langle f^s_{ab:},f^t_{kl:} \rangle^2 }}$$  
The denominator performs a normalization
operation with the effect of down-weighing ambiguous
matches, by penalizing features from one image which have
multiple highly-rated matches in the other image.  
## Geometric transformation estimation  
Done by transformation regression CNN:
$$G: {\Bbb R}^{h\times w\times h\times w}\rightarrow {\Bbb R}^K$$  
Where $$K$$ are the degrees of freedom (depending on if affine transformation...).  

## Soft-inlier count  
Inspired by RANSAC.  
It sums matching scores over all possible matches. 
RANSAC formulates an hypotesis for all the possible cases and takes as valid the most scored one.  
![ransac](/images/papers/ransac.PNG)  
The RANSAC algorithm is the following one:  
$$c_R=\sum m(p_i)$$ where $$m(p)=     1 \text{ if } d(p,l)\lt \text{t }   0 \text{ otherwise}.  $$  
The problem is this algorithm is not differentiable since you are setting strict values based on a threshold. Thus, they reformulate the algorithm as the sum of the match scores over all possible matches penalized by the RANSAC algorithm. This way differentiation is taken into account by the sum and gradients are adjusted by the coefficient each term is multiplied by.  
The new algorithm:  
$$c=\sum s_{ij}m_{ij}$$  
$$m_{ij}$$ are the same as in the RANSAC meanwhile $$s_{ij}$$ are the sum of the matched scores.  


