##### Reference
1. [Prevalence of Neural Collapse during the terminal phase of deep learning training](https://arxiv.org/abs/2008.08186)
2. [Understanding Imbalanced Semantic Segmentation Through Neural Collapse](https://arxiv.org/abs/2301.01100)
3. [Combating Representation Learning Disparity with Geometric Harmonization](https://arxiv.org/abs/2310.17622)
4. [Exploring Deep Neural Networks via Layer-Peeled Model: Minority Collapse in Imbalanced Training](https://arxiv.org/abs/2101.12699)
##### Language Models
Twitter
  - https://x.com/_jasonwei/status/1660709485005144064
  - https://x.com/ShayneRedford/status/1660670375834058754/photo/1

Other Reference
- [[Manifold {Concept}]]

---
#### Prior Knowledge: 
- [[Do DL models overfit?]]
#### Neural Collapse
- It is a phenomenon observed in the "deep" neural network during first stage of training, specially in clarification task. As network approaches zero training loss, it enters a structured stated called Neural Collapse; where several remarkable geometric properties emerge in the learned feature space.
- This structure is called Simplex ETF (Equiangular Tight Frame)
- The characteristics of Neural Collapse are 
	- [NC1] Variability Collapse: within class variability collapse, i.e. data points within the same class have nearly identical feature representations, clustering around a single vector (class mean)
	- [NC2] Simplex Equiangular Tight Frame (Simplex ETF): class mean arrange themselves in a geometrically optimal structure where class means are equally space and maximally separated from each other. This configuration minimises classification error by maximising distance between classes
	- [NC3] Self Duality: class-mean and linear classification (used to map features to class labels) converge to same structure. This alignment is known as self-duality and more symmetric decision boundary
	- [NC4] Nearest Class Center Simplification (NCC): each data point is classified by finding nearest class-mean in terms of euclidean distance

#### Neural Collapse in Semantic Segmentation
##### Neural Collapse in SS vs Classification
- Symmetric ETF of NC does not hold in semantic segmentation for both feature centers and classifiers. This is because 
	- Classes in classification have low correlation while in semantic segmentation are contextually correlated 
	- NC observed in image recognition task relies on balance class distribution of training dataset and is broken when data imbalance emerges [ref 4]
- Solution: A center regularisation branch to extract feature center of each semantic class. We Regularise them by another clasifier layer with fixed ETF layer. The fixed classifier forces feature centers to be aligned with the appealing structure of maximal discriminativity and equiangular separation
	![[Pasted image 20241102004806.png]]

##### Main Approach
- Lemma 1: Regularising feature centers in segmentation model into simplex ETF relieves imbalance dilemma for semantic segmentation


	$$\mathbf{M} = \sqrt{\frac{K}{K - 1}} \, \mathbf{U} \left( \mathbf{I}_K - \frac{1}{K} \mathbf{1}_K \mathbf{1}_K^{\top} \right)$$
	
	
			$\forall \qquad K$: number of classes in task (seg, cls)


			$\sqrt{\frac{K}{K - 1}}$: scaling factor to maintain consistent length across all weight vetors
			

			$U, \ ( \mathbf{I}_K ), \ ( \mathbf{I}_K )$ are rotation vector, identity vector (K x K) and all one vector of dim-K


- Center Collapse Regularisation
	- The framework is of two parts: **point/pixel recognition branch** and **center regularization branch** 
	- In this branch, $\mathbf{z}_i$ (feature vector) of each class and compute $\mathbf{z}_k$ (feature center) to generate center labels ($\mathbf{y}_k$) of all classes based on ground truth y
		
		$$\overline{z}_k = \frac{1}{n_k} \sum_{y_i = k}^{n_k} z_i, \quad \overline{y}_k = y_i = k \qquad \forall \qquad \mathbf{n}_k \text{ is the number of samples in Z belonging to k-th class})$$
- Final Loss function 
	1. CR loss: $\mathcal{L}_{\text{CR}}(\overline{\mathbf{Z}}, \mathbf{W}^*) = - \sum_{k=1}^{K} \log \left( \frac{\exp(\overline{\mathbf{z}}_k^{\top} \mathbf{w}_k^*)}{\sum_{k'=1}^{K} \exp(\overline{\mathbf{z}}_{k'}^{\top} \mathbf{w}_{k'}^*)} \right).$
	2. Primary Loss (Pixel-wise CE): $\mathcal{L}_{PR}(Z, y) = - \sum_{i} \sum_{k=1}^{K} y_{i,k} \log \left( p_{i,k} \right)$
	3. Total Loss: $\mathcal{L}_{\text{total}} = \mathcal{L}_{\text{PR}}(\mathbf{Z}, \mathbf{y}) + \lambda \mathcal{L}_{\text{CR}}(\overline{\mathbf{Z}}, \mathbf{W}^*)$

##### Theoretical Support
1. Gradient wrt center classifier

		$$\frac{\partial \mathcal{L}_{\text{CR}}}{\partial \mathbf{w}_k} = \left( p_k \left( \overline{\mathbf{z}}_k \right) - 1 \right) \overline{\mathbf{z}}_k + \sum_{k' \neq k}^{K-1} p_k \left( \overline{\mathbf{z}}_{k'} \right) \overline{\mathbf{z}}_{k'},$$
		
		$$\qquad = \underbrace{\sum_{y_i = k}^{n_k} \left( p_k \left( \overline{\mathbf{z}}_k \right) - 1 \right) \frac{\mathbf{z}_i}{n_k}}_{\text{within-class}} \underbrace{\sum_{k' \neq k}^{K - 1} \sum_{y_j = k'}^{n_k} p_k \left( \overline{\mathbf{z}}_{k'} \right) \frac{\mathbf{z}_j}{n_{k'}}}_{\text{between-class}}$$
	+ This equation is imbalanced and decomposes into 2 part: within-class and between-class. Hence, the gradients of some minor classes can be likely swallowed by other classes. This can be resolved if the weights of center classifier are fixed
2. Gradient wrt Pixel feature
	- ETF structured classifier enables the gradient transfer between minor classes and obtains better discrimination among minor classes. Thus it avoids the features of minor classes converging in a close position