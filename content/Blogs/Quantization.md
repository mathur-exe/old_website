**Memory Needed to store / run a model**
$\text{memory} = \frac{\text{nr\_bits}}{8} \times \text{nr\_params}$
- Memory to store a 70B model at different precisions:
	1. **64-bits** = $\frac{64}{8} \times 70B \approx 560 \, \text{GB}$
	2. **32-bits** = $\frac{32}{8} \times 70B \approx 280 \, \text{GB}$
	3. **16-bits** = $\frac{16}{8} \times 70B \approx 140 \, \text{GB}$

**Symmetric Quantization**
- The range of the original floating-point values is mapped to a symmetric range around zero in the quantized space. The highest absolute value ($\alpha$) is used for linear mapping around 0.
- Formula for linear mapping around zero:

|                                                                |                                                                                |
| :------------------------------------------------------------: | ------------------------------------------------------------------------------ |
|                $s = \frac{2^{b-1} - 1}{\alpha}$                | $\forall \qquad b: \text{number of bits, } \alpha: \text{high absolute value}$ |
| $x_{\text{quantized}} = \text{round} \left( s \cdot x \right)$ | $\forall \qquad x: \text{input}$                                               |
|          $x_{\text{dequantized}} = \frac{[value]}{s}$          | Dequantization to FP32                                                         |

![[Pasted image 20241023212217.png]]

- For example:
	- $s = 127 / 10.8 = 11.76$
	- $x_{\text{quantized}} = \text{round}(11.76 \cdot x)$ 

**Asymmetric Quantization**
- The scaling is based on the min-max value, and as a result, the 0-value is shifted.
- Here, [-127, 128] represents the min-max value of INT-8:

|                                                                  |                                                                                    |
| :--------------------------------------------------------------: | ---------------------------------------------------------------------------------- |
|            $s = \frac{128 - (-127)}{\alpha - \beta}$             | $\forall \qquad s: \text{ scale factor}$                                           |
|     $Z = \text{round}\left(-s \cdot \beta \right) - 2^{b-1}$     | $\forall \qquad Z: \text{zero poin, } (\alpha, \beta): \text{min-max value in FP}$ |
| $X_{\text{quantized}} = \text{round}\left(s \cdot X + Z \right)$ |                                                                                    |
|   $X_{\text{dequantized}} = \frac{(\text{some value} - Z)}{s}$   | Dequantization                                                                     |

![[Pasted image 20241023212126.png]]
- For example:
	- $s = \frac{255}{10.8 - (-7.59)} = 13.86$
	- $Z = \text{round}(-13.86 \cdot -7.59) - 128 = -23$
	- $X_{\text{quantized}} = \text{round}(13.86 \cdot X + -23)$

**Range Mapping, Clipping and Calibration**
- Issue with above approaches: outliers. Hence, we'll clip certain values.
- If we were to map the full range of this vector, all small values would get mapped to the same lower-bit representation and lose their differentiating factor.

> $Y = w.x + b$
> 	- Biases are only a few million $\rightarrow$ higher precision such as INT16.
> 	- Weights {billions} $\rightarrow$ being higher in number, the main effect of quantization is applied to the weights.
> 	- Activation $\rightarrow$ updated after each hidden layer, and values are only known during inference.

- Broadly, methods of calibrating quantization of weights and activation:
	1. Post-Training Quantization (PTQ)
	2. Quantization Aware Training (QAT)
		- Quantization **during** training/fine-tuning
### Post-Training Quantization
- Weight and activation are quantized post training. 
- Quantization weights --> sym or asym
- Quantizatio Activation --> Dynamic / Statuc

| Dynamic Quantization                                                                                                                     | Static Quantization                                                                                                                             |
| :--------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------- |
| - This quantization is repeated for each every layer, hence, every layer has it's own separate z & s. Thus different quantization scheme | - instead of calculating for each layer, a calibration dataset is used by model to collect distribution and calculate ==*global*== Z & s values |
| ``` calculate activation > calculate Z & s > quantize ```                                                                                | ```calibration dataset > calculate Z & s > quantization```                                                                                      |

> Going below 4-bit quantization is not advised as the quantization error increases with every lossing bit
> 	1. GPTQ (full mode on GPU)
> 	2. GGUF (offload layer on CPU)
#### GPTQ
- used in practice to quantize to 4 bit
- uses asymm quant and does so layer-by-layer such that each layer is independent of other

```
layer weights to inverse-Hessian > quantize (FP32 to INT4) > dequantize (INT4 to FP32) > calculate weighted quantization error > quantization
```

|                             |                                     |
| --------------------------- | ----------------------------------- |
| $q = \frac{X_1 - X_1}{h_1}$ | Hessian-weighted quantization error |
| $X_2 = X_2 + q \cdot h_2$   | Updated weights                     |
- For Example:
	![[Pasted image 20241023212411.png]]
	![[Pasted image 20241023212336.png]]
	
	1. q = (0.5 - 0.297)/0.2 = 0.203
	2. x2 = 0.3 + 0.203 * 0.8 =  0.4624
	
> Original Paper: https://arxiv.org/pdf/2210.17323
> 
> YouTube video: https://www.youtube.com/watch?v=mii-xFaPCrA
#### GGUF
- Why GGUF? when we donot have enough VRAM to run full LLM on GPU despite GPTQ
```
Weight matrix > super block > sub block > sub block quantization > super block quantization
```
- Absmax Quantization: 
	- Basic Formula: $X_{\text{quantized}} = S \cdot X$
	- Scale Factor: $S$ is the max absolute value of wt in the block

```
Matrix:
W = [ [ 0.5, -0.2,  0.8, -0.6 ],
      [-0.1,  0.4, -0.7,  0.3 ],
      [ 0.9, -0.4,  0.2, -0.5 ],
      [ 0.7,  0.1, -0.3,  0.6 ] ]

Super Block 1: [[ 0.5, -0.2,  0.8, -0.6 ],
				[-0.1,  0.4, -0.7,  0.3 ]]

Super Block 2: [ [ 0.9, -0.4,  0.2, -0.5 ],
				  [ 0.7,  0.1, -0.3,  0.6 ] ]

Sub Block 1: [[ 0.5, -0.2 ],
			  [-0.1,  0.4 ] ]
> Quantized Sub Block 1 = (1 / 0.5) * (Sub Block 1)
                      = [[ 1, -0.4 ],
                        [-0.2,  0.8 ]]

Sub Block 2: [[ 0.8, -0.6 ],
			  [-0.7,  0.3 ] ]
> Quantized Sub Block 2 = (1 / 0.8) * (Sub Block 2)
                      = [ [ 1, -0.75 ],
                          [-0.875, 0.375 ] ]

Sub Block 3: [[ 0.9, -0.4 ],
			  [ 0.7,  0.1 ] ]
> Quantized Sub Block 3 = (1 / 0.9) * (Sub Block 3)
                      = [ [ 1, -0.44 ],
                          [ 0.78,  0.11 ] ]


Sub Block 4: [[ 0.2, -0.5 ],
			  [-0.3,  0.6 ] ]
> Quantized Sub Block 4 = (1 / 0.6) * [(Sub Block 4)
                      = [ [ 0.33, -0.83 ],
                          [-0.5,   1 ] ]

Qunatization Super Block 1: 1
Qunatization Super Block 2: 1

Final Matrix:
W_quantized = [ [  1,  -0.4,    1,  -0.75  ],
                [-0.2,   0.8, -0.875,  0.375 ],
                [  1,  -0.44,  0.33,  -0.83  ],
                [0.78,  0.11,  -0.5,     1   ] ]
```

### Quantization Aware Training
==still studying==

##### Question
Q] If the distance between two values on a number scale is large, it should imply greater precision, as there is more room for additional values between them.
+ Floating-point numbers are not uniformly distributed across the number line. Instead, they are densely packed near zero and become sparser as the magnitude increases. This is because: 
	1. **Exponent Influence**: 
		- The exponent part determines the scale of the number. For numbers close to zero, the exponent is highly negative, allowing for very fine granularity.
		- For example: 
			Exponent bit: 00100 $\rightarrow 2^2$
			Exponent value: exponent - bias = 4 - 15 = -11
			Exponent = $2^{-11} = 0.00048828$
	1. **Mantissa Precision**: When the exponent is smaller (i.e., closer to zero), the mantissa represents smaller increments.
- **Addressing Intuition**:
	1. Floating-point numbers are in a logarithmic scale.
	2. **Density Near Zero**: There are **more representable numbers per unit interval** near zero, meaning the spacing between consecutive numbers is **smaller**, leading to **higher absolute precision**.
	3. **Sparser as Magnitude Increases**: As numbers get larger, the spacing between consecutive representable numbers increases, resulting in **lower absolute precision** but maintaining **consistent relative precision**.
	
Q] What's the point of quantization and then dequantization? 
- This process allows us to calculate quantization error. Essentially we are creating a weighted-quantization error based on importance of weight

Q] Significance of inverse- Hessian Matrix?

**Reference**
- https://newsletter.maartengrootendorst.com/p/a-visual-guide-to-quantization
- 
