---
title: Mixed-Precision 
layout: default
permalink: 404.html
---

# Introduction
Deep Neural Networks (DNNs) have lead to breakthroughs in Computer Vision, Natural Language Understanding, Speech Recognition tasks and many others.  
</br>  
With the increasing size of a deep neural networks (typically improves accuracy ), the computational resources grow up (GPU utilization , Memory) so new techniques have been developed to train DNNs faster without losing accuracy or modifying the network hyper-parameters.  
</br>  
By lowering the required memory will be enabled to train larger models or train with larger minibatches.  
</br>
In neural nets, all the computations are done in single precision floating point arithmetic.  
**single precision floating point arithmetic**  deals with 32 bit floating point numbers which means that all the floats in all the arrays that represent inputs, activations, weights .. etc are 32-bit floats (FP32).  
</br>  
So an idea to reduce memory usage by dealing with 16-bits floats which called **half precision floating point format**.    
The problem in **half precision** is that it had a smaller range and lower precision than single or double precision floats, and for this reason half precision won't achieve the same accuracy.  


