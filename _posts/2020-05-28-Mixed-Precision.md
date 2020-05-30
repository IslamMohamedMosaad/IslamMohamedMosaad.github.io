---
layout: post
title: "Mixed Precision Training"
author: "Islam Mohamed"
categories: journal
tags: [documentation,sample]
---

# A High-Level Look

Deep Neural Networks (DNNs) have achieved breakthroughs in a number of areas, including Computer Vision, Natural Language Understanding, Speech Recognition tasks, and many others.  
  
Although increasing network size typically improves accuracy, the computational resources also increase (GPU utilization, Memory) so new techniques have been developed to train models faster without losing accuracy or modifying the network hyper-parameters by lowering the required memory will enable us to train larger models or train with larger mini-batches.  
  
In neural nets, all the computations are done in a **single precision floating point**.  
  
**Single precision floating point** arithmetic deals with 32-bit floating point numbers which means that all the floats in all the arrays that represent inputs, activations, weights .. etc are 32-bit floats (FP32).  
  
So an idea to reduce memory usage by dealing with 16-bits floats which called **Half precision floating point format** (FP16).  
The problem in Half precision localized in its smaller range and lower precision, unlike single or double precision, floats, and for this reason, half precision sometimes won’t able to achieve the same accuracy.  
  
But with **Mixed precision** which uses both single and half precision representations will able to speed up training and achieving the same accuracy.  

# Problems with half precision  
To understand the problems with half precision, let’s have a look what an FP16 looks like : 

![half precision floating point format]({{'' | relative_url }})
{: style="width: 550px; max-width: 100%;"}
*Fig. 1 : half precision floating point formatt.*

**It divided into three modules:**
1. The bit number 15 is the sign bit.
2. the bists for 10 to 14  are the exponent.
3. the final 10 bits are the fraction.


**The value for this representation is calculated as shown below**  
![half precision floating point format  with value for bits]({{'' | relative_url }})
{: style="width: 550px; max-width: 100%;"}
*Fig. 2 : half precision floating point formatt with value for bits.

1. If the exponent bits is ones (11111), then the value will be NaN ("Not a number").  
2. If the exponent bits is zeros (0000), then the value will be a subnormal number and calculated by :  
                    ```  (−1) ^ signbit × 2 ^  −14 × 0.0 + binary fraction series ```  
3. Otherwise the value will be a normalized value and calculated by :  
                    ```  (−1) ^ signbit × 2 ^ exponent value − 15 × 1.0 + binary fraction series ```  

> Based on half precision floating point methodology if we tried to add 1 + 0.0001 the output will be 1 because of the limited range and aligning between 1 and 0.0001 as shown in this [answer](https://cs.stackexchange.com/questions/63642/how-to-add-two-numbers-in-iee754-half-precision-format).  
> And that will cause numbers of problems while training DNNs.  
> For trying and investigation through conversion or adding in binary 16 float point check this [site](http://weitz.de/ieee/).

#The main issues while training with FP16
1. Values is imprecise.
2. Underflow Risk.
3. Exploding Risk.

## Values is imprecise
In neural Network training all weights, activations, and gradients are stored as FP16.  
And as we know updating weights is done based on this equation   
**New_weight = Weight - Learning_Rate * Weight.Gradient**  
Since Weight.Gradient and Learning_Rate usually with small values and as shown before in half precision if the weight is 1 and Learning_Rate is 0.0001 or lower that will made freezing thrugh weights value.  

## Underflow Risk
In FP16, Gradients will get converted to zero because gradients usually are too low.   
In FP16 arithmetic the values smaller than 0.000000059605 = 2^24 become zero as this value is the smallest positive subnormal number and for more details investigate [here](https://en.wikipedia.org/wiki/Half-precision_floating-point_format).   
With underflow, network never learns anything.  

## Overflow Risk
In FP16, activations and network paramters can increase till hitting NANs.   
With overflow or exploding, network learns garbage.  




