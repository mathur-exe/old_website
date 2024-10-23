**Reference**
- https://newsletter.maartengrootendorst.com/p/a-visual-guide-to-quantization

**Memory Needed to store / run a model**
$$\text{memory} = \frac{\text{nr\_bits}}{8} \times \text{nr\_params}$$
- Memory to store a 70B model at different precisions
	1. $\textbf{64-bits} = \frac{64}{8} \times 70B \approx 560 \, \text{GB}$
	2. $\textbf{32-bits} = \frac{32}{8} \times 70B \approx 280 \, \text{GB}$
	3. $\textbf{16-bits} = \frac{16}{8} \times 70B \approx 140 \, \text{GB}$

**Symmetric Quantization**
- the range of the original floating-point values is mapped to a symmetric range around zero in the quantized space. The highest absolute value ($\alpha$) is used for linear mapping around 0
- Formula for linear mapping around zero, 
$$
s = \frac{2^{b-1} - 1}{\alpha} \, \qquad|  \qquad \, x_{\text{quantized}} = \text{round} \left( s \cdot x \right) \qquad| \qquad \forall \qquad \text{b: number of bit, } \alpha \text{: high absolute value, x: input}
$$
	For example:
		s = 127 / 10.8 = 11.76
		x_quantized = round(11.76 x ___ )
- To retrive original FP32 value
$$x_{\text{dequantized}} = \frac{\textcolor{red}{\blacksquare \blacksquare \blacksquare \blacksquare \blacksquare \blacksquare \blacksquare}}{s}$$
**Asymmetric Quantization**
- The scaling is based on min-max value and as result 0-value is also shifted
- Here, [-127, 128] represents the min-max value of INT-8
$$  
s = \frac{128 - (-127)}{\alpha - \beta} \qquad| \qquad Z = \text{round}\left(-s \cdot \beta \right) - 2^{b-1} \qquad| \qquad X_{\text{quantized}} = \text{round}\left(s \cdot X + Z \right)$$
$$\forall \qquad s: \text{scale factor, } Z: \text{zeropoint}, (\alpha, \beta) \text{ : min-max value in FP}$$
	`For example:`
	`s = 255/(10.8 - (-7.59)) = 13.86`
	`Z = round(-13.86 * -7.59) - 128 = -23`
	`X_quantized = round(13.86 * ___ * -23)`

- To dequantize: $X_{\text{dequantized}} = \frac{(\text{some value} - Z)}{S}$

**Range Mapping, Clipping and Calibration**
- Issue with above approaches: outlier. Hence, we'll clip certain values
- If we were to map the full range of this vector, all small values would get mapped to the same lower-bit representation and lose their differentiating factor

$$Y = w.x + b$$
- bias are only a few million --> higher precision such as INT16
- weight {billions} --> being in higher number, the main effect of quantization is put on weights. 
- activation --> updated after each hidden layer and value are only know during inference 

	Broadly methods of calibrating quantization of weights and activation 
	1. Post-Training Quantization (PTQ)
	2. Quantization Aware Training (QAT)
		- Quantization _**during**_ training/fine-tuning

### Post-Training Quantization

### Quantization Aware Training
##### Question
Q] If the distance between two values on a number scale is large, it should imply greater precision, as there is more room for additional values between them.
+ Floating-point numbers are not uniformly distributed across the number line. Instead, they are densely packed near zero and become sparser as the magnitude increases. This is because: 
	1. Exponent Influence: 
		- The exponent part determines the scale of the number. For numbers close to zero, the exponent is highly negative, allowing for very fine granularity.
		- For example: 
			Exponent bit: 00100 --> 2^2
			Exponent value: exponent - bias = 4 - 15 = -11
			Exponent = 2^-11 = 0.00048828
	1. Mantissa Precision: When the exponent is smaller (i.e., closer to zero), the mantissa represents smaller increments.
- Addressing Intuition
	1. FP is in logarithmic scale
	2. Density Near Zero: There are **more representable numbers per unit interval** near zero, meaning the spacing between consecutive numbers is **smaller**, leading to **higher absolute precision**.
	3. Sparser as Magnitude Increases: As numbers get larger, the spacing between consecutive representable numbers increases, resulting in **lower absolute precision** but maintaining **consistent relative precision**.

