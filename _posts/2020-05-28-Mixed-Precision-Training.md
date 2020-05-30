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

![half precision floating point format]({{'/assets/img/floating-point-arithmetic-half-precision.jpg' | relative_url }})
{: style="width: 550px; max-width: 100%;"}
*Fig. 1 : half precision floating point formatt.*

**It divided into three modules:**
1. The bit number 15 is the sign bit.
2. the bists for 10 to 14  are the exponent.
3. the final 10 bits are the fraction.


**The value for this representation is calculated as shown below**  
![half precision floating point format  with value for bits]({{'assets/img/1*2SF0OFMsC606KSDaVtBAdg.png' | relative_url }})
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

# The main issues while training with FP16
1. Values is imprecise.
2. Underflow Risk.
3. Exploding Risk.

### Values is imprecise
In neural Network training all weights, activations, and gradients are stored as FP16.  
And as we know updating weights is done based on this equation   
**New_weight = Weight - Learning_Rate * Weight.Gradient**  
Since Weight.Gradient and Learning_Rate usually with small values and as shown before in half precision if the weight is 1 and Learning_Rate is 0.0001 or lower that will made freezing thrugh weights value.  

### Underflow Risk
In FP16, Gradients will get converted to zero because gradients usually are too low.   
In FP16 arithmetic the values smaller than 0.000000059605 = 2^-24 become zero as this value is the smallest positive subnormal number and for more details investigate [here](https://en.wikipedia.org/wiki/Half-precision_floating-point_format).   
With underflow, network never learns anything.  

### Overflow Risk
In FP16, activations and network paramters can increase till hitting NANs.   
With overflow or exploding, network learns garbage.  

# The Proposed Techniques for Training with Mixed Precision  
Mainly there are three techniques for preventing the loss of critical information.  
1. Single precision FP32 Master copy of weights and updates.  
2. Loss (Gredient) Scaling.  
3. Accumulating half precision products into single precision.  

### Single precision FP32 Master copy of weights and updates
To overcome the first problem we use a copy from the FP32 master of all weights and in each iteration apply the forward and backward propagation in FP16 and then update weights stored in the master copy as shown below.  

![Mixed precision training iteration for a layer]({{'assets/img/Mixed precision training iteration for a layer.png' | relative_url }})
{: style="width: 550px; max-width: 100%;"}
*Fig. 3 : Mixed precision training iteration for a layer.*

Through the storing an additional copy of weights increases the memory requirements but the overall memory consumptions is approximately halved the need by FP32 training.  

### Loss (Gredient) Scaling
* Gradient values with magnitudes below 2^-27 were not relevant to training network, whereas it was important to preserve values in the [2^-27, 2^-24] range.  
* Most of the half precision range is not used by gradients, which tend to be small values with magnitudes below 1. Thus, we can multiply them by a scale factor S to keep relevant gradient values from becoming zeros.  
* This constant scaling factor is chosen empirically or, if gradient statistics are available, directly by choosing a factor so that its product with the maximum absolute gradient value is below 65,504 (the maximum value representable in FP16).  
* Of course we don’t want those scaled gradients to be in the weight update, so after converting them into FP32, we can divide them by this scale factor (once they have no risks of becoming 0).   


### Accumulating half precision products into single precision
After investigatin through last issue found that the neural network arithmetic operations falls into three groups vector dot-products , Reductions and point-wise operations.  
These categories benefit from different treatment when it comes to re-duced precision arithmetic.  
* Some networks require that the FP16 vector dot-product accumulates the partial products into an FP32 value, which is then converted to FP16 before storing.  
* Large reductions (sums across elements of a vector) should be carried out in FP32. Such reductionsmostly  come  up  in  batch-normalization  layers  when  accumulating  statistics  and  softmax  layers.  
* Point-wise  operations,  such  as  non-linearities  and  element-wise  matrix  products,  are  memory-bandwidth limited. Since arithmetic precision does not impact the speed of these operations, eitherFP16 or FP32 math can be used.  




