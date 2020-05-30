---
title: Mixed-Precision 
layout: default
permalink: 404.html
---


Deep Neural Networks (DNNs) have achieved a breakthroughs in Computer Vision, Natural Language Understanding, Speech Recognition tasks and many others.  
  
Although increasing network size typically improves accuracy but the computational resources also increase (GPU utilization , Memory) so new techniques have been developed to train models faster without losing accuracy or modifying the network hyper-parameters.  
By lowering the required memory will enable us to train larger models or train with larger minibatches.
  
In neural nets, all the computations are done in single precision floating point.
  
**Single precision floating point arithmetic**  deals with 32 bit floating point numbers which means that all the floats in all the arrays that represent inputs, activations, weights .. etc are 32-bit floats (FP32).  
   
So an idea to reduce memory usage by dealing with 16-bits floats which called **Half precision floating point format**.    
The problem in **Half precision** localized in its smaller range and lower precision unlike single or double precision floats, and for this reason half precision sometimes won't able to achieve the same accuracy.  

But with **Mixed precision** which use both single and half precision representations will able to speed up training and achieving the same accuracy.



