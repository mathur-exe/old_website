![[Pasted image 20241003195732.png]]
Reference: https://x.com/SamuelMLSmith/status/1838617836543185091

> Questions to ponder (link)
>	1. Why don't heavily parameterized neural networks overfit the data?
>	2. What is the effective number of parameters?
>	3. Why doesn't backpropagation head for a poor local minima?
>	4. When should one stop the backpropagation and use the current parameters?

---
⃣ Adding Table of content for this page

---
#### Paper: [[Visualising Loss Landscape of Neural Nets]]
==TODO: Complete Notes==

#### Theory
- if there is a loss, but a loss gradient is zero for a given weight, then the algorithm doesn't change for this weight. This implies though, for the network to stay in a local minima, every weight and bias has to itself be in a local minima with respect to derivative of loss

###### Loss function: Complexity vs Non-linearity
> A large number of parameters and nonlinearity is neither sufficient nor necessary for a complex loss function. If you have billions of parameters and the loss is computed by individually squaring and summing them, GD will converge to the gloval minimum, despite nonlinearity and high parameter count. Non-convexity is the real issue.

👆There is a distinction between the complexity of a loss function and its nonlinearity, as well as the challenges posed by nonconvexity in optimization problems*

1. While the high parameter count makes the model expensive (able to learn complex features), it's doesn't necessarily make the optimization problem difficult or loss function complexity. The structure of the loss function and its curvature plays a more significant role in optimization difficulty.
2. Although non-linearity allows the model capture complex info, it's doesn't make optimization landscape difficult to navigate
3. REAL CHALLENGE: Nonconvexity
	- Multiple local minima
	- Saddle Points
	- Flat Regions, gradien --> 0 making progress slow
	- In DL, loss landscape is nonconvex due to [[#Complex Interactions in DL]] between layer, wts, non-linear activation function
###### Complex Interactions in DL
1. Hierarchical transformations
	- one layer’s output becomes the next layer’s input, lead to intricate relationships between parameters
1. Nonlinearity from Activation Functions
	- small changes in ip / wt can lead to disproportionate changes in loss function (because of nonlinearity of activation function)


#### Challenges in Neural Network Optimisation (ref: [Deep Learning Book]([deeplearningbook.org/contents/optimization.html](https://www.deeplearningbook.org/contents/optimization.html)
==TODO: Read==

